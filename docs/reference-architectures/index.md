---
title: Architectures de référence Azure
description: Architectures de référence, schémas et des recommandations préconisées de mise en œuvre pour les charges de travail courantes dans Azure.
layout: LandingPage
ms.topic: landing-page
ms.openlocfilehash: 4c0006404ed0c0f603e9599cd9c04e6d02fdbfeb
ms.sourcegitcommit: a5e549c15a948f6fb5cec786dbddc8578af3be66
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/06/2018
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="azure-reference-architectures"></a>Architectures de référence Azure

Nos architectures de référence sont organisées par scénario, avec les architectures connexes regroupées. Chaque architecture comprend les pratiques recommandées, ainsi que des considérations pour l’extensibilité, la disponibilité, la facilité de gestion et la sécurité. La plupart incluent également une solution pouvant être déployée.

<section class="series">
    <ul class="panelContent">

<!-- N-tier -->
<li style="display: flex; flex-direction: column;">
    <a href="./n-tier/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./n-tier/images/n-tier-sql-server.svg" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application multi-niveau</h3>
                        <p>Déployer une application multiniveau dans Azure, pour Windows ou Linux.</p>
                        <p>Les configurations sont affichées pour SQL Server et Apache Cassandra. Pour la haute disponibilité, déployez une configuration actif/passif dans deux régions.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>

<!-- Hybrid network -->
<li style="display: flex; flex-direction: column;">
    <a href="./hybrid-networking/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./hybrid-networking/images/vpn.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Réseau hybride</h3>
                        <p>Cette série présente des options pour la création d’une connexion réseau entre un réseau local et Azure.</p>
                        <p>Les configurations incluent des VPN de site à site ou Microsoft Azure ExpressRoute pour une connexion privée et dédiée.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Network DMZ -->
<li style="display: flex; flex-direction: column;">
    <a href="./dmz/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./dmz/images/secure-vnet-dmz.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Zone DMZ réseau</h3>
                        <p>Cette série montre comment créer une zone DMZ réseau pour protéger la limite entre un réseau virtuel Azure et un réseau local ou Internet.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Identity management -->
<li style="display: flex; flex-direction: column;">
    <a href="./identity/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./identity/images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Gestion des identités</h3>
                        <p>Cette série montre les options pour l’intégration de votre environnement local Active Directory (AD) avec un réseau Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- App Service web application -->
<li style="display: flex; flex-direction: column;">
    <a href="./app-service-web-app/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./app-service-web-app/images/scalable-web-app.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Application web App Service</h3>
                        <p>Cette série présente les meilleures pratiques pour les applications web qui utilisent Azure App Service.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
    <!-- Jenkins build server -->
<li style="display: flex; flex-direction: column;">
    <a href="./jenkins/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./jenkins/images/logo.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Serveur de builds Jenkins</h3>
                        <p>Déployez et utilisez un serveur Jenkins professionnel et évolutif sur Azure.</p>
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
                            <img src="./sharepoint/images/sharepoint.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Batterie de serveurs SharePoint Server 2016</h3>
                        <p>Déployez et exécutez une batterie de serveurs SharePoint Server 2016 sur Azure avec des groupes de disponibilité SQL Server AlwaysOn.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- SAP NetWeaver and SAP HANA -->
<li style="display: flex; flex-direction: column;">
    <a href="./sap/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./sap/images/sap.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>SAP NetWeaver et SAP HANA</h3>
                        <p>Déployez et exécutez SAP NetWeaver et SAP HANA dans un environnement à haute disponibilité dans Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
</ul>