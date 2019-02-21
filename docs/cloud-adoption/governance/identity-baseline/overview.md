---
title: 'Framework d’adoption du cloud : Présentation de la discipline Base de référence des identités'
titleSuffix: Microsoft Cloud Adoption Framework for Azure
ms.service: architecture-center
ms.subservice: enterprise-cloud-adoption
ms.custom: governance
ms.date: 02/11/2019
description: Raison de la Base de référence des identités dans le cadre de la gouvernance du cloud
author: BrianBlanchard
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 14569b8db68434ee30fa7bff7fba2da5fa537049
ms.sourcegitcommit: 273e690c0cfabbc3822089c7d8bc743ef41d2b6e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/08/2019
ms.locfileid: "55901102"
---
# <a name="caf-identity-baseline-discipline-overview"></a>Framework d’adoption du cloud : Présentation de la discipline Base de référence des identités

La Base de référence des identités est l’une des [cinq disciplines de la gouvernance du cloud](../governance-disciplines.md) dans le [ Modèle de gouvernance du Framework d’adoption du cloud](../overview.md). L’identité est de plus en plus considérée comme le périmètre de sécurité principal dans le cloud, ce qui constitue un changement par rapport à l’approche traditionnelle de la sécurité réseau. Les services d’identité fournissent les mécanismes de base de contrôle d’accès et d’organisation au sein des environnements informatiques, et la discipline Base de référence des identités vient en complément de la [discipline Base de référence de la sécurité](../security-baseline/overview.md) en implémentant de manière cohérente les exigences d’authentification et d’autorisation aux tâches d’adoption du cloud.

> [!NOTE]
> La gouvernance Base de référence des identités ne remplace pas les équipes, les processus et les procédures informatiques existants qui permettent à votre entreprise de gérer et de sécuriser les services d’identité. L’objectif principal de cette discipline est d’identifier les risques métier potentiels liés aux identités et de fournir des conseils sur l’atténuation des risques au personnel informatique responsable de l’implémentation, de la gestion et de l’exploitation de votre infrastructure de gestion des identités. Lorsque vous élaborez des stratégies et des processus de gouvernance, veillez à faire participer les équipes informatiques pertinentes à vos processus de planification et de révision du code.

Cette section des lignes directrices du Framework d’adoption du cloud (CAF) décrit l’approche à adopter pour élaborer une discipline Base de référence des identités dans le cadre de votre stratégie de gouvernance cloud. Ce guide s’adresse principalement aux architectes cloud de votre organisation et aux autres membres de votre équipe de gouvernance cloud. Cependant, les décisions, les stratégies et les processus qui découlent de cette discipline doivent impliquer un engagement et des discussions avec les membres concernés des équipes informatiques responsables de l’implémentation et de l’administration de vos solutions de gestion des identités.

Si votre organisation manque d’expertise interne en matière de sécurité et de Base de référence des identités, envisagez de faire appel à des consultants externes pour y remédier. Envisagez également de faire appel aux [Services de conseil Microsoft](https://www.microsoft.com/enterprise/services), au service d’adoption du cloud [Microsoft FastTrack](https://azure.microsoft.com/programs/azure-fasttrack) ou à des experts externes pour aborder les préoccupations liées à cette discipline.

## <a name="policy-statements"></a>Policy statements

Les déclarations de stratégie exploitables et les exigences d’architecture qui en résultent constituent la base d’une discipline Base de référence des identités. Pour prendre connaissance des exemples de déclarations de stratégie, consultez l’article [Déclarations de stratégie Base de référence des identités](./policy-statements.md). Ces exemples peuvent servir de point de départ à l’élaboration de stratégies de gouvernance de votre organisation.

> [!CAUTION]
> Les exemples de stratégies émanent de l’expérience commune des clients. Pour mieux corréler ces stratégies aux besoins spécifiques de gouvernance cloud, exécutez les étapes suivantes pour créer des déclarations de stratégie qui répondent aux besoins particuliers de votre entreprise.

## <a name="developing-identity-baseline-governance-policy-statements"></a>Élaboration de déclarations de stratégie de gouvernance Base de référence des identités

Les six étapes suivantes offrent des exemples et des options potentielles à prendre en compte lors de l’élaboration de la gouvernance Base de référence des identités. Chaque étape peut servir de thème de discussion au sein de votre équipe de gouvernance et avec les équipes concernées et les équipes informatiques de votre organisation en vue d’établir les stratégies et processus nécessaires à l’atténuation des risques liés aux identités.

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
                        <h3>Modèle Base de référence des identités</h3>
                        <p class="x-hidden-focus">Télécharger le modèle pour documenter une discipline Base de référence des identités</p>
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
                        <p class="x-hidden-focus">Appréhendez les motivations et les risques généralement associés à la discipline Base de référence des identités.</p>
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
                        <p class="x-hidden-focus">Indicateurs permettant de déterminer à quel moment il est pertinent d’investir dans la discipline Base de référence des identités.</p>
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
                        <h3>Développer des processus d’adhérence à la politique</h3>
                        <p class="x-hidden-focus">Suggestions de processus pour appuyer la conformité aux stratégies dans la discipline Base de référence des identités.</p>
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
                        <p class="x-hidden-focus">Services Azure qui peuvent être implémentés pour soutenir la discipline Base de référence des identités.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

<!-- markdownlint-enable MD033 -->

## <a name="next-steps"></a>Étapes suivantes

Commencez par évaluer les [risques d’affaires](./business-risks.md) dans un environnement spécifique.

> [!div class="nextstepaction"]
> [Comprendre les risques commerciaux](./business-risks.md)
