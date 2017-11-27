---
title: "Étend la fonctionnalité des modèles Azure Resource Manager"
description: "Décrit des conseils et astuces pour étendre la fonctionnalité des modèles Azure Resource Manager"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: d9cad6cfbfe8d42bcb21ef9e77504b80aef2d694
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="extend-azure-resource-manager-template-functionality"></a>Étend la fonctionnalité des modèles Azure Resource Manager

En 2016, l’équipe de modèles et pratiques AzureCAT a créé un ensemble de [modèles de blocs de construction](https://github.com/mspnp/template-building-blocks/wiki) Azure Resource Manager afin de simplifier le déploiement de ressources. Chaque bloc de construction contient un ensemble de modèles prédéfinis qui déploient des ensembles de ressources spécifiés par des fichiers de paramètres distincts.

Les modèles de blocs de construction sont conçus pour être combinés afin de créer des déploiements plus volumineux et complexes. Par exemple, le déploiement d’une machine virtuelle dans Azure nécessite un réseau virtuel, des comptes de stockage et d’autres ressources. Le [modèle de bloc de construction de réseau virtuel](https://github.com/mspnp/template-building-blocks/wiki/VNet-(v1)) déploie un réseau virtuel et des sous-réseaux. Le [modèle de bloc de construction de machine virtuel](https://github.com/mspnp/template-building-blocks/wiki/Windows-and-Linux-VMs-(v1)) déploie des comptes de stockage, des interfaces réseau et des machines virtuelles réelles. Vous pouvez ensuite créer un script ou un modèle pour appeler les deux modèles de bloc de construction et leurs fichiers de paramètres correspondants, pour déployer une architecture complète en une seule opération.

En développant les modèles de bloc de construction, l’équipe de modèles et pratiques a conçu plusieurs concepts pour étendre la fonctionnalité de modèle Azure Resource Manager. Dans cette série, nous examinerons plusieurs de ces concepts afin que vous puissiez les utiliser dans vos propres modèles.

> [!NOTE]
> Ces articles requièrent une connaissance approfondie des modèles Azure Resource Manager.