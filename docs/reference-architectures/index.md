---
title: "Architectures de référence Azure"
description: "Architectures de référence, schémas et des recommandations préconisées de mise en œuvre pour les charges de travail courantes dans Azure."
layout: LandingPage
NOTE: edit the template in ./build/reference-architectures !!!
ms.openlocfilehash: e513e2545fb95b486b11a067479e879f4abd4a8f
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/14/2018
---
# <a name="azure-reference-architectures"></a>Architectures de référence Azure

Nos architectures de référence sont organisées par scénario, avec les architectures connexes regroupées. Chaque architecture comprend les pratiques recommandées, ainsi que des considérations pour l’extensibilité, la disponibilité, la facilité de gestion et la sécurité. La plupart incluent également une solution pouvant être déployée.

<section class="series">
    <ul class="panelContent">
    <!-- Windows VM workloads -->
<li style="display: flex; flex-direction: column;">
    <a href="./virtual-machines-windows/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./virtual-machines-windows/images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Charges de travail de machine virtuelle Windows</h3>
                        <p>Cette série commence avec les meilleures pratiques pour l’exécution d’une machine virtuelle Windows unique, puis les machines virtuelles à charge équilibrée et enfin une application multiniveau et multirégion.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Linux VM workloads -->
<li style="display: flex; flex-direction: column;">
    <a href="./virtual-machines-linux/index.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./virtual-machines-linux/images/n-tier.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Charges de travail de machine virtuelle Linux</h3>
                        <p>Cette série commence avec les meilleures pratiques pour l’exécution d’une seule machine virtuelle Linux unique, puis les machines virtuelles à charge équilibrée et enfin une application multiniveau et multirégion.</p>
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