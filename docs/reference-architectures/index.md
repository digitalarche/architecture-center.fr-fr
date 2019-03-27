---
title: "Architectures de référence\_Azure"
description: Architectures de référence et conseils d’implémentation pour les charges de travail courantes dans Azure.
layout: LandingPage
ms.topic: landing-page
ms.date: 03/07/2019
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="azure-reference-architectures"></a>Architectures de référence Azure

Nos architectures de référence sont organisées par scénario. Chaque architecture comprend les pratiques recommandées, ainsi que des considérations pour l’extensibilité, la disponibilité, la facilité de gestion et la sécurité. La plupart d’entre elles comprend aussi une implémentation de référence ou de solution déployable.

Passer à : [IA](#ai-and-machine-learning) | [Big data](#big-data-solutions) | [IoT](#internet-of-things) | [Microservices](#microservices) | [Serverless](#serverless-applications) | [Réseaux virtuels](#virtual-networks) | [Charges de travail de machine virtuelle](#vm-workloads) | [SAP](#sap) | [Active Directory](#extend-on-premises-active-directory-to-azure) | [Applications web](#web-applications)

<!-- markdownlint-disable MD033 -->

## <a name="ai-and-machine-learning"></a>IA et Machine Learning

<ul  class="panelContent cardsF">
<!-- Distributed training of deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/training-deep-learning.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/batch-ai.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Entraînement distribué des modèles d’apprentissage profond</h3>
                        <p>Effectuez un entraînement distribué des modèles d’apprentissage profond sur les clusters des machines virtuelles avec processeur.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Training of Python scikit-learn models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/training-python-models.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Entraînement des modèles scikit-learn Python</h3>
                        <p>Pratiques recommandées pour le réglage des hyperparamètres d’un modèle Python scikit-learn.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring for deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-deep-learning.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/batch-ai.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Scoring par lots pour les modèles d’apprentissage profond</h3>
                        <p>Automatiser les programmes de traitement par lots qui appliquent un transfert de style neural vers une vidéo.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring of Python models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-python.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Scoring par lots des modèles Python</h3>
                        <p>Procédez au scoring par lots de nombreux modèles Python en parallèle selon une planification à l’aide d’Azure Machine Learning.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Batch scoring of Spark models on Azure Databricks -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/batch-scoring-databricks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/databricks.png" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Scoring par lots de modèles Spark sur Azure Databricks</h3>
                        <p>Créez une solution scalable pour le scoring par lots d’un modèle de classification Apache Spark avec Azure Databricks.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Real-time scoring of Python and deep learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-python.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/python-powered-h.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Scoring en temps réel des modèles Python et d’apprentissage profond</h3>
                        <p>Déployez des modèles de Python en tant que services web pour effectuer des prédictions en temps réel, à l’aide de modèles de Python réguliers ou de modèles d’apprentissage profond.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Real-time scoring of R machine learning models -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/realtime-scoring-r.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/logo-r.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Scoring en temps réel des modèles Machine Learning R</h3>
                        <p>Implémentez un service de prédiction en temps réel en R en exécutant Microsoft Machine Learning Server dans Azure Kubernetes Service (ACS).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Real-time recommendation API -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/real-time-recommendation.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/machine-learning.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>API de recommandation en temps réel</h3>
                        <p>Entraînez un modèle de recommandation à l’aide d’Azure Databricks et déployez-le comme API à l’aide d’Azure Machine Learning.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Enterprise-grade conversational bot -->
<li style="display: flex; flex-direction: column;">
    <a href="./ai/conversational-bot.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/bot-services.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Bot de conversation de classe Entreprise</h3>
                        <p>Comment créer un bot de conversation de classe Entreprise avec Azure Bot Framework.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="big-data-solutions"></a>Solutions Big Data

<ul  class="panelContent cardsF">
<!-- Enterprise BI with SQL Data Warehouse -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-sqldw.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Enterprise BI avec SQL Data Warehouse</h3>
                        <p>Pipeline ELT (extract-load-transform) permettant de déplacer des données à partir d’une base de données locale dans SQL Data Warehouse.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Automated enterprise BI with Azure Data Factory -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/enterprise-bi-adf.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/data-guide.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>BI d’entreprise automatisée avec Azure Data Factory</h3>
                        <p>Automatisez un pipeline ELT pour effectuer le chargement incrémentiel d’une base de données locale.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Stream processing with Azure Databricks -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-databricks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/databricks.png" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Traitement de flux de données avec Azure Databricks</h3>
                        <p>Pipeline de traitement de flux qui joint des enregistrements à partir de deux flux, enrichit le résultat et calcule une moyenne mobile.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Stream processing with Azure Stream Analytics -->
<li style="display: flex; flex-direction: column;">
    <a href="./data/stream-processing-stream-analytics.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-analysis-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Traitement de flux de données avec Azure Stream Analytics</h3>
                        <p>Pipeline de traitement de flux de bout en bout qui met en corrélation les enregistrements issus de deux flux de données pour calculer une moyenne mobile.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="internet-of-things"></a>Internet des Objets

<ul  class="panelContent cardsF">
<!-- Azure IoT reference architecture -->
<li style="display: flex; flex-direction: column;">
    <a href="./iot/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./iot/_images/iot.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Architecture de référence Azure IoT</h3>
                        <p>Architecture recommandée pour les applications IoT sur Azure utilisant des composants PaaS (Platform-as-a-Service).</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="microservices"></a>Microservices

<ul  class="panelContent cardsF">
<!-- Microservices on Azure Kubernetes Service (AKS) -->
<li style="display: flex; flex-direction: column;">
    <a href="./microservices/aks.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/aks.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Microservices sur Azure Kubernetes Service (AKS)</h3>
                        <p>Architecture recommandée pour déployer des microservices sur AKS.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Microservices architecture on Azure Service Fabric -->
<li style="display: flex; flex-direction: column;">
    <a href="./microservices/service-fabric.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sf.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Architecture des microservices sur Azure Service Fabric</h3>
                        <p>Architecture recommandée pour les microservices sur Service Fabric.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="serverless-applications"></a>Applications serverless

<ul  class="panelContent cardsF">
<!-- Serverless web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application web serverless</h3>
                        <p>Une application web serverless gère le contenu statique à partir du stockage Blob Azure et implémente une API à l’aide d’Azure Functions.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Event processing using Azure Functions -->
<li style="display: flex; flex-direction: column;">
    <a href="./serverless/event-processing.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/functions.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Traitement des événements à l’aide d’Azure Functions</h3>
                        <p>Une architecture basée sur les événements qui ingère un flux de données et utilise Functions pour traiter les données.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="virtual-networks"></a>Réseaux virtuels

<ul  class="panelContent cardsF">
<!-- Hybrid network using a virtual private network (VPN) -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/vpn.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Réseau hybride utilisant un réseau privé virtuel (VPN)</h3>
                        <p>Connectez un réseau local à un réseau virtuel Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid network using ExpressRoute -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Réseau hybride utilisant ExpressRoute</h3>
                        <p>Utilisez une connexion privée et dédiée pour étendre un réseau local à Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hybrid network using ExpressRoute with VPN failover -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/expressroute-vpn-failover.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/expressroute.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Réseau hybride utilisant ExpressRoute avec basculement VPN</h3>
                        <p>Utilisez ExpressRoute avec une connexion de basculement via un VPN, pour la haute disponibilité.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub-spoke network topology -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/hub-spoke.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Topologie de réseau hub-and-spoke</h3>
                        <p>Le hub est un point central de connectivité pour votre réseau local, tout en isolant les charges de travail.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Hub-spoke topology with shared services -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/shared-services.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/gateway.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Topologie hub-and-spoke avec des services partagés</h3>
                        <p>Étendez une topologie hub-and-spoke en incluant les services partagés tels qu’Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- DMZ between Azure and on-premises -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-hybrid.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Zone DMZ entre Azure et les réseaux locaux</h3>
                        <p>Utilisez des appliances virtuelles réseau pour créer un réseau hybride sécurisé.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- DMZ between Azure and the Internet -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/secure-vnet-dmz.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Zone DMZ entre Azure et Internet</h3>
                        <p>Utilisez des appliances virtuelles réseau pour créer un réseau sécurisé qui accepte le trafic Internet.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Highly available network virtual appliances -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/nva-ha.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/vnet.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Appliances réseau virtuelles hautement disponible</h3>
                        <p>Déployez un ensemble d’appliances virtuelles réseau pour la haute disponibilité dans Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="vm-workloads"></a>Charges de travail de machine virtuelle

<ul  class="panelContent cardsF">
<!-- N-tier application with SQL Server -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application multiniveau avec SQL Server</h3>
                        <p>Machines virtuelles configurées pour une application multiniveau à l’aide de SQL Server sur Windows.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Multi-region N-tier application -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/multi-region-sql-server.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/windows.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application multiniveau multirégion</h3>
                        <p>Application multiniveau dans deux régions pour la haute disponibilité, à l’aide des groupes de disponibilité AlwaysOn SQL Server.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- N-tier application with Cassandra -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/n-tier-cassandra.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/linux-penguin.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application multiniveau avec Cassandra</h3>
                        <p>Machines virtuelles configurées pour une application multiniveau à l’aide d’Apache Cassandra sur Linux.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/jenkins.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Serveur de builds Jenkins</h3>
                        <p>Serveur Jenkins de classe Entreprise et évolutif sur Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SharePoint Server 2016 farm -->
<li style="display: flex; flex-direction: column;">
    <a href="./sharepoint/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Batterie de serveurs SharePoint Server 2016</h3>
                        <p>Batterie de serveurs SharePoint Server 2016 sur Azure, hautement évolutive, avec des groupes de disponibilité SQL Server AlwaysOn.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="sap"></a>SAP

<ul  class="panelContent cardsF">
<!-- SAP NetWeaver -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-netweaver.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver</h3>
                        <p>SAP NetWeaver sur Windows, dans un environnement à haute disponibilité qui prend en charge la récupération d’urgence.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SAP S/4HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/sap-s4hana.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP S/4HANA</h3>
                        <p>SAP S/4HANA sur Windows, dans un environnement à haute disponibilité qui prend en charge la récupération d’urgence.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- SAP HANA on Azure Large Instances -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/hana-large-instances.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP HANA sur Azure (grandes instances)</h3>
                        <p>Les grandes instances HANA sont déployées sur des serveurs physiques dans des régions Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="extend-on-premises-active-directory-to-azure"></a>Étendre Active Directory en local à Azure

<ul  class="panelContent cardsF">
<!-- Integrate with Azure Active Directory -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/azure-active-directory.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Intégrer à Azure Active Directory</h3>
                        <p>Intégrez ses domaines AD locaux avec Azure Active Directory.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Extend an on-premises Active Directory domain to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Étendre un domaine Active Directory local à Azure</h3>
                        <p>Déployez Active Directory Domain Services (AD DS) dans Azure pour étendre votre domaine local.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Create an AD DS forest in Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Créer une forêt AD DS dans Azure</h3>
                        <p>Créez dans Azure un domaine AD distinct, approuvé par votre forêt AD locale.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Extend Active Directory Federation Services (AD FS) to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/active-directory-vm.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Étendre les services de fédération Active Directory (AD FS) dans Azure</h3>
                        <p>Utilisez AD FS pour l’authentification et l’autorisation fédérée des composants s’exécutant dans Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>

## <a name="web-applications"></a>Applications web

<ul  class="panelContent cardsF">
<!-- Basic web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/basic-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application web de base</h3>
                        <p>Application web qui utilise Azure App Service et Azure SQL Database.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Highly scalable web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/scalable-web-app.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application web hautement évolutive</h3>
                        <p>Pratiques optimisées d’amélioration de l’évolutivité dans une application web.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Highly available web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/multi-region.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application web hautement évolutive</h3>
                        <p>Exécutez une application web App Service dans plusieurs régions pour permettre une haute disponibilité.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
<!-- Web application monitoring on Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/app-monitoring.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="../_images/icons/app-service.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Supervision des applications web sur Azure</h3>
                        <p>Surveillez une application web hébergée dans Azure App Service.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>