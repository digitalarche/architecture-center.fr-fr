---
title: 'Framework d’adoption du cloud : Présentation de la discipline Cohérence des ressources'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Présentation de la discipline Cohérence des ressources
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: ce29efab1502d59513528ab4b640173346aee516
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55900890"
---
# <a name="caf-resource-consistency-discipline-overview"></a>Framework d’adoption du cloud : Présentation de la discipline Cohérence des ressources

La Cohérence des ressources est l’une des [cinq disciplines de la gouvernance du cloud](../governance-disciplines.md) dans le [Modèle de gouvernance du framework d’adoption du cloud](../overview.md). Elle se concentre sur les moyens à utiliser pour établir des stratégies en lien avec la gestion opérationnelle d’un environnement, d’une application ou d’une charge de travail. Les équipes informatiques s’occupent souvent de la surveillance des applications, des charges de travail et des performances des ressources. En général, ils exécutent également les tâches nécessaires pour répondre aux demandes de mise à l’échelle, corriger les violations de SLA et prévenir de façon proactive les violations de contrat de niveau de service de performances via la correction automatisée. Parmi les cinq disciplines de gouvernance cloud, la Cohérence des ressources permet de garantir que les ressources sont configurées de manière cohérente, de sorte qu’elles sont détectables par le service informatique, sont incluses dans les solutions de récupération et peuvent être intégrées à des processus opérationnels reproductibles.

> [!NOTE]
> La gouvernance Cohérence des ressources ne remplace pas les équipes, les processus et les procédures informatiques existants qui permettent à votre organisation de gérer efficacement les ressources basées sur le cloud. L’objectif principal de cette discipline est d’identifier les risques métier potentiels et de fournir des conseils sur l’atténuation des risques au personnel informatique responsable de la gestion de vos ressources dans le cloud. Lorsque vous élaborez des stratégies et des processus de gouvernance, veillez à faire participer les équipes informatiques pertinentes à vos processus de planification et de révision.

Cette section du framework d’adoption du cloud explique comment développer une discipline Cohérence des ressources dans le cadre de votre stratégie de gouvernance cloud. Ce guide s’adresse principalement aux architectes cloud de votre organisation et aux autres membres de votre équipe de gouvernance cloud. Cependant, les décisions, les stratégies et les processus qui découlent de cette discipline doivent impliquer un engagement et des discussions avec les membres concernés des équipes informatiques responsables de l’implémentation et de l’administration de vos solutions de cohérence des ressources.

Si votre organisation manque d’expertise interne en matière de stratégies Cohérence des ressources, envisagez de faire appel à des consultants externes pour y remédier. Envisagez également de faire appel aux [Services de conseil Microsoft](https://www.microsoft.com/enterprise/services), au service d’adoption du cloud [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) ou à des experts de l’adoption du cloud externes pour aborder les thèmes relatifs à l’organisation, au suivi et à l’optimisation de vos ressources basées sur le cloud.

## <a name="policy-statements"></a>Policy statements

Les instructions de stratégie exploitables et les exigences d’architecture qui en résultent constituent le fondement d’une discipline Cohérence des ressources. Pour prendre connaissance des exemples d’instructions de stratégie, consultez l’article [Instructions de stratégie Cohérence des ressources](./policy-statements.md). Ces exemples peuvent servir de point de départ à l’élaboration de stratégies de gouvernance de votre organisation.

> [!CAUTION]
> Les exemples de stratégies émanent de l’expérience commune des clients. Pour mieux corréler ces stratégies aux besoins spécifiques de gouvernance cloud, exécutez les étapes suivantes afin de créer des instructions stratégiques qui répondent aux besoins particuliers de votre entreprise.

## <a name="developing-resource-consistency-governance-policy-statements"></a>Développer des instructions stratégiques pour la gouvernance Cohérence des ressources

Les six étapes suivantes offrent des exemples et des options potentielles à prendre en compte lors de l’élaboration de la gouvernance Cohérence des ressources. Chaque étape peut servir de thème de discussion au sein de votre équipe de gouvernance et avec les équipes concernées et les équipes informatiques de votre organisation en vue d’établir les stratégies et processus nécessaires à l’atténuation des risques liés à la Cohérence des ressources.

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
                        <h3>Modèle de Cohérence des ressources</h3>
                        <p class="x-hidden-focus">Télécharger le modèle pour documenter une discipline Cohérence des ressources</p>
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
                        <h3>Risques commerciaux</h3>
                        <p class="x-hidden-focus">Appréhendez les motivations et les risques généralement associés à la discipline Cohérence des ressources.</p>
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
                        <p class="x-hidden-focus">Indicateurs permettant de déterminer à quel moment il est pertinent d’investir dans la discipline Cohérence des ressources.</p>
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
                        <p class="x-hidden-focus">Suggestions de processus pour appuyer la conformité aux stratégies dans la discipline Cohérence des ressources.</p>
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
                        <p class="x-hidden-focus">Services Azure qui peuvent être implémentés pour soutenir la discipline Cohérence des ressources.</p>
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
