---
title: Building microservices on Azure (Création de microservices sur Azure)
description: Conception, génération et exploitation d’architectures de microservices sur Azure
ms.date: 03/07/2019
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: e74d6f6098eb68c8bbd737cf3a047ce5de04327d
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58346142"
---
# <a name="building-microservices-on-azure"></a>Building microservices on Azure (Création de microservices sur Azure)

<!-- markdownlint-disable MD033 -->

<img src="../_images/microservices.svg" style="float:left; margin-top:8px; margin-right:8px; max-width: 80px; max-height: 80px;"/>

Les microservices sont un style architectural répandu pour générer des applications résilientes, hautement scalables, capables d’évoluer rapidement et pouvant être déployées indépendamment. Mais une architecture de microservices réussie nécessite une approche différente pour concevoir et générer des applications.

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./introduction.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Que sont les microservices ?</h3>
                        <p>En quoi les microservices diffèrent d’autres architectures, et quand devez-vous les utiliser ?</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../guide/architecture-styles/microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Style d’architecture de microservices</h3>
                        <p>Vue d’ensemble globale du style d’architecture de microservices</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="examples-of-microservices-architectures"></a>Exemples d’architectures de microservices

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/infrastructure/service-fabric-microservices.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Utiliser Service Fabric pour décomposer des applications monolithiques</h3>
                        <p>Approche itérative de la décomposition d’un site web ASP.NET en microservices.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../example-scenario/data/ecommerce-order-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Traitement évolutif des commandes sur Azure</h3>
                        <p>Traitement des commandes à l’aide d’un modèle de programmation fonctionnel implémenté par le biais de microservices.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="build-a-microservices-application"></a>Générer une application de microservices

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./model/domain-analysis.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Utiliser l’analyse de domaine pour modéliser les microservices</h3>
                        <p>Pour éviter certains pièges lors de la conception de microservices, utilisez l’analyse de domaine pour définir les limites de vos microservices.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Architecture de référence pour Azure Kubernetes Services (AKS)</h3>
                        <p>Cette architecture de référence montre une configuration AKS de base qui peut être le point de départ de la plupart des déploiements.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="../reference-architectures/microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Architecture de référence pour Azure Service Fabric</h3>
                        <p>Cette architecture de référence montre une configuration recommandée qui peut être le point de départ de la plupart des déploiements.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Concevoir une architecture de microservices</h3>
                        <p>Ces articles approfondissent les connaissances relatives à la façon de générer une application de microservices, basée sur une implémentation de référence qui utilise Azure Kubernetes Services (AKS).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./design/patterns.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Modèles de conception</h3>
                        <p>Ensemble de modèles de conception utiles pour les microservices.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="operate-microservices-in-production"></a>Exploiter des microservices en production

<ul  class="panelContent cardsZ">
<li style="display: flex; flex-direction: column;">
    <a href="./logging-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Enregistrement et surveillance</h3>
                        <p>La nature distribuée des architectures de microservices rend la journalisation et la supervision particulièrement critique.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<li style="display: flex; flex-direction: column;">
    <a href="./ci-cd.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardText">
                        <h3>Intégration et déploiement continus</h3>
                        <p>L’intégration continue et la livraison continue (CI/CD) sont essentielles à la réussite des microservices.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>
