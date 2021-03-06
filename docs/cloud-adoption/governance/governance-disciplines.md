---
title: 'Framework d’adoption du cloud : Les cinq disciplines de la gouvernance cloud'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Vue d’ensemble du contenu de la gouvernance dans le framework d'adoption du cloud
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ddd7f5e4b7e79ebe2ddf87be7421b6c4691aced1
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/17/2019
ms.locfileid: "59639867"
---
# <a name="the-five-disciplines-of-cloud-governance"></a>Les cinq disciplines de la gouvernance cloud

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsI">
<li style="display: flex; flex-direction: column;">
    <div class="cardSize">
        <div class="cardPadding" style="padding-bottom:10px;">
            <div class="card" style="padding-bottom:10px;">
                <div class="cardText" style="padding-left:0px;">
Toute modification apportée à des processus métier ou à des plates-formes technologiques présente un risque. Les équipes de gouvernance cloud, dont les membres sont parfois appelés les gardiens du cloud, sont chargées de minimiser ces risques tout en évitant d’interrompre l’adoption et l’innovation.<br/><br/>Le modèle de gouvernance CAF Guide ces décisions (quel que soit la plateforme de cloud choisi) en se concentrant sur le développement de la stratégie d’entreprise et <a href="#disciplines-of-cloud-governance">Disciplines de gouvernance Cloud de</a>. <a href="./journeys/overview.md">Parcours actionnables</a> illustrent ce modèle à l’aide des services Azure. Cet article fait office de page d'accueil pour les cinq disciplines du modèle de gouvernance CAF.
                </div>
            </div>
        </div>
    </div>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../_images/operational-transformation-govern-highres.png" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize">
            <div class="cardPadding" style="padding-bottom:10px;">
                <div class="card" style="padding-bottom:10px;">
                    <div class="cardText" style="padding-left:0px;">
<img src="../_images/operational-transformation-govern-highres.png" alt="Diagram of the CAF governance model: Corporate policy and governance disciplines">
<br>
<i>Figure 1 : Diagramme de la stratégie d’entreprise et les Disciplines de gouvernance de Cloud cinq</i>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="disciplines-of-cloud-governance"></a>Disciplines de gouvernance cloud

Sur chaque fournisseur de cloud, il existe des disciplines gouvernance courantes qui peuvent servir de guide pour aider à informer les stratégies et aligner les chaînes d’outils. Ces disciplines orientent les décisions en matière de niveau d'automatisation et de mise en œuvre de la stratégie d’entreprise entre fournisseurs de cloud.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsA">
<li style="display: flex; flex-direction: column;">
    <a href="./cost-management/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/cost-management.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cost Management</h3>
                        <p>Les coûts constituent une préoccupation majeure pour les utilisateurs du cloud. Développez des stratégies de contrôle des coûts pour toutes les plateformes cloud.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./security-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/security-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Base de référence de la sécurité</h3>
                        <p>La sécurité est un sujet complexe et propre à chaque entreprise. Une fois les exigences de sécurité définies, les stratégies de gouvernance cloud sont appliquées aux configurations réseau, données et ressources.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./identity-baseline/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/identity-baseline.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Base de référence des identités</h3>
                        <p>En matière d'application des exigences d'identité, les incohérences peuvent augmenter le risque de violation. La discipline Base de référence des identités se concentre sur les moyens de veiller à ce que l'identité soit appliquée de manière cohérente à toutes les tâches d'adoption du cloud.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./resource-consistency/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/resource-consistency.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Cohérence des ressources</h3>
                        <p>Les opérations cloud dépendent d'une configuration cohérente des ressources. Les outils de gouvernance permettent de configurer de manière cohérente les ressources afin d'atténuer les risques d'intégration, de détectabilité et de récupération.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./deployment-acceleration/overview.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/governance/deployment-acceleration.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Accélération du déploiement</h3>
                        <p>La centralisation, la standardisation et la cohérence des approches de déploiement et de configuration améliorent les pratiques de gouvernance. Lorsqu'elles sont proposées via des outils de gouvernance cloud, elles créent un facteur cloud qui permet d'accélérer les activités de déploiement.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->
