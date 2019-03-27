---
title: 'Framework d’adoption du cloud : Vue d’ensemble de la discipline Accélération du déploiement'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Raison de l’accélération du déploiement en raison de la gouvernance du cloud.
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 8a9cd359f631708d07ab601c4488b969dc8294a0
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58246590"
---
# <a name="caf-deployment-acceleration-discipline-overview"></a>Framework d’adoption du cloud : Vue d’ensemble de la discipline Accélération du déploiement

L’accélération du déploiement est l’une des [cinq disciplines de la gouvernance du cloud](../governance-disciplines.md) dans le [ Modèle de gouvernance du Framework d’adoption du cloud](../overview.md). Cette discipline met l’accent sur les moyens d’établir des stratégies pour régir la configuration ou le déploiement des ressources. Dans les cinq disciplines de la gouvernance du cloud, l’accélération du déploiement inclut le déploiement, l’alignement de la configuration et la réutilisabilité des scripts. Il peut s’agir d’activités manuelles ou d’activités DevOps entièrement automatisées. Dans les deux cas, les stratégies sont en grande partie similaires. Au fur et à mesure que cette discipline mûrit, l’équipe de gouvernance du cloud peut jouer le rôle de partenaire dans les DevOps et les stratégies de déploiement en accélérant les déploiements et en éliminant les freins à l’adoption du cloud, grâce à la mise en œuvre de ressources réutilisables.

Cet article décrit le processus d’accélération du déploiement qu’une entreprise expérimente durant les phases de planification, de construction, d’adoption et d’exploitation d’une solution cloud. Il est impossible pour tout un document de prendre en compte les exigences d’une organisation. En tant que telle, chaque section de cet article décrit les activités minimales et potentielles suggérées. L’objectif initial de ces activités est de vous aider à créer un [MVP de stratégie](../policy-compliance/overview.md#minimum-viable-product-mvp-for-policy) mais aussi à établir une infrastructure pour une évolution de [stratégie incrémentielle](../policy-compliance/overview.md#incremental-policy-growth). L’équipe de gouvernance du cloud doit décider combien investir dans ces activités pour améliorer l’Accélération du déploiement.

> [!NOTE]
> La discipline Accélération du déploiement ne remplace pas les équipes, les processus et les procédures informatiques existants qui permettent à votre entreprise de déployer et de configurer efficacement les ressources cloud. L’objectif principal de cette discipline est d’identifier les risques métier potentiels et de fournir des conseils sur l’atténuation des risques au personnel informatique responsable de la gestion de vos ressources dans le cloud. Lorsque vous élaborez des stratégies et des processus de gouvernance, veillez à faire participer les équipes informatiques pertinentes à vos processus de planification et de révision du code.

Ce guide s’adresse principalement aux architectes cloud de votre organisation et aux autres membres de votre équipe de gouvernance cloud. Cependant, les décisions, les stratégies et les processus qui découlent de cette discipline doivent impliquer un engagement et des discussions avec les membres concernés dans votre entreprise et vos équipes informatiques, en particulier les responsables du déploiement et de la configuration des charges de travail dans le cloud.

## <a name="policy-statements"></a>Policy statements

Les déclarations de stratégie exploitables et les exigences d’architecture qui en résultent constituent la base d’une discipline Accélération du déploiement. Pour prendre connaissance des exemples de déclarations de stratégie, consultez l’article [Déclarations de stratégie d’accélération du déploiement](./policy-statements.md). Ces exemples peuvent servir de point de départ à l’élaboration de stratégies de gouvernance de votre organisation.

> [!CAUTION]
> Les exemples de stratégies émanent de l’expérience commune des clients. Pour mieux corréler ces stratégies aux besoins spécifiques de gouvernance cloud, exécutez les étapes suivantes afin de créer des déclarations de stratégie qui répondent aux besoins particuliers de votre entreprise.

## <a name="developing-deployment-acceleration-governance-policy-statements"></a>Élaboration de déclarations de stratégie de gouvernance Accélération du déploiement

Les six étapes suivantes vous aideront à définir des stratégies de gouvernance pour contrôler le déploiement et la configuration des ressources dans votre environnement cloud.

<!-- markdownlint-disable MD033 -->

<ul class="panelContent cardsE">
<li style="display: flex; flex-direction: column;">
    <a href="./template.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-template.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Accélération du déploiement</h3>
                        <p class="x-hidden-focus">Télécharger le modèle pour documenter une discipline Accélération du déploiement</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li><li style="display: flex; flex-direction: column;">
    <a href="./business-risks.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-risks.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Risques d’affaires</h3>
                        <p class="x-hidden-focus">Appréhendez les motivations et les risques généralement associés à la discipline Accélération du déploiement.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./metrics-tolerance.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-metrics.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Indicateurs et métriques</h3>
                        <p class="x-hidden-focus">Indicateurs permettant de déterminer à quel moment il est pertinent d’investir dans la discipline Accélération du déploiement.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./compliance-processes.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-enforce.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Développer des processus d’adhésion à la stratégie</h3>
                        <p class="x-hidden-focus">Suggestions de processus pour appuyer la conformité aux stratégies dans la discipline Accélération du déploiement.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./discipline-improvement.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-maturity.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Maturité</h3>
                        <p class="x-hidden-focus">Adéquation de la maturité de la gestion du cloud avec les phases d’adoption du cloud.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./toolchain.md">
        <div class="cardSize">
            <div class="cardPadding" >
                <div class="card" >
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../../_images/governance/process-toolchain.png" class="x-hidden-focus"/>
                        </div>
                    </div>
                    <div class="cardText" style="padding-left:0px;">
                        <h3>Chaîne d’outils</h3>
                        <p class="x-hidden-focus">Services Azure qui peuvent être implémentés pour soutenir la discipline Accélération du déploiement.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="next-steps"></a>Étapes suivantes

Commencez par évaluer les [risques commerciaux](./business-risks.md) dans un environnement spécifique.

> [!div class="nextstepaction"]
> [Comprendre les risques commerciaux](./business-risks.md)

<!-- markdownlint-enable MD033 -->