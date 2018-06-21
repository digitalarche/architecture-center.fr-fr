---
title: Gestion des identités
description: Explique et compare les différentes méthodes disponibles pour la gestion des identités dans les systèmes hybrides couvrant la limite local/cloud avec Azure.
layout: LandingPage
ms.openlocfilehash: de98ee30306f5e712759ab7140bd430cb6d4cd75
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 02/23/2018
ms.locfileid: "29478236"
---
<!-- This file is generated! -->
<!-- See the templates in ./build/reference-architectures  -->
<!-- See data in index.json -->

# <a name="identity-management"></a>Gestion des identités

Ces architectures de référence affichent les options pour l’intégration de votre environnement local Active Directory (AD) avec un réseau Azure. <br/>[Quelle solution choisir ?](./considerations.md)

<section class="series">
    <ul class="panelContent">
    <!-- Integrate with Azure Active Directory -->
<li style="display: flex; flex-direction: column;">
    <a href="./azure-ad.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/azure-ad.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Intégrer à Azure Active Directory</h3>
                        <p>Intégrez les forêts et domaines locaux de Active Directory avec Azure AD.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD DS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-extend-domain.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-extend-domain.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Étendre AD DS à Azure</h3>
                        <p>Étendez votre environnement Active Directory à Azure à l’aide de Active Directory Domain Services.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Create an AD DS forest in Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adds-forest.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adds-forest.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Créer une forêt AD DS dans Azure</h3>
                        <p>Créez un domaine AD distinct dans Azure qui est approuvé par des domaines de votre forêt locale.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    <!-- Extend AD FS to Azure -->
<li style="display: flex; flex-direction: column;">
    <a href="./adfs.md" style="display: flex; flex-direction: column; flex: 1 0 auto;">
        <div class="cardSize" style="flex: 1 0 auto; display: flex;">
            <div class="cardPadding" style="display: flex;">
                <div class="card">
                    <div class="cardImageOuter">
                        <div class="cardImage">
                            <img src="./images/adfs.svg" height="140px" />
                        </div>
                    </div>
                    <div class="cardText">
                        <h3>Étendre AD FS à Azure</h3>
                        <p>Utilisez Active Directory Federation Services pour l’authentification fédérée et l’autorisation dans Azure.</p>
                    </div>
                </div>
            </div>
        </div>
    </a>
</li>
    </ul>
</section>

<ul class="panelContent cardsI">
</ul>