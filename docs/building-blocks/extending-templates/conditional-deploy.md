---
title: "Déployer une ressource de manière conditionnelle dans un modèle Azure Resource Manager"
description: "Explique comment étendre les fonctionnalités des modèles Azure Resource Manager au déploiement conditionnel d’une ressource en fonction de la valeur d’un paramètre"
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: e911e7dc41b4f71ebfaf13a00f8cdbb5b4e2578b
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a>Déployer une ressource de manière conditionnelle dans un modèle Azure Resource Manager

Certains scénarios nécessitent que vous conceviez votre modèle pour le déploiement d’une ressource reposant sur une condition spécifique, telle que la présence ou l’absence d’une valeur de paramètre. Par exemple, votre modèle peut déployer un réseau virtuel et inclure des paramètres pour spécifier d’autres réseaux virtuels à des fins d’homologation. Si vous n’avez spécifié aucune valeur de paramètre pour l’homologation, vous ne souhaitez pas que Resource Manager déploie la ressource d’homologation.

Pour effectuer cette opération, utilisez [l’élément `condition`][azure-resource-manager-condition] dans la ressource afin de tester la longueur de votre tableau de paramètres. Si cette longueur est égale à zéro, renvoyez la valeur `false` pour empêcher le déploiement ; pour toutes les longueurs supérieures à zéro, renvoyez la valeur `true` afin d’autoriser le déploiement.

## <a name="example-template"></a>Exemple de modèle

Examinons un exemple de modèle illustrant cette approche. Notre modèle utilise [l’élément `condition`][azure-resource-manager-condition] pour contrôler le déploiement de la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`. Cette ressource crée une homologation entre deux réseaux virtuels Azure dans la même région.

Examinons chaque section du modèle.

L’élément `parameters` définit un paramètre unique nommé `virtualNetworkPeerings` : 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualNetworkPeerings": {
      "type": "array",
      "defaultValue": []
    }
  },
```
Notre paramètre `virtualNetworkPeerings` est un `array` et présente le schéma suivant :

```json
"virtualNetworkPeerings": [
    {
        "remoteVirtualNetwork": {
            "name": "my-other-virtual-network"
        },
        "allowForwardedTraffic": true,
        "allowGatewayTransit": true,
        "useRemoteGateways": false
    }
]
```

Les propriétés de notre paramètre spécifient les [paramètres associés à l’homologation de réseaux virtuels][vnet-peering-resource-schema]. Nous fournissons les valeurs de ces propriétés lorsque nous spécifions la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` dans la section `resources` :

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "virtualNetworks"
      ],
      "copy": {
          "name": "iterator",
          "count": "[length(variables('peerings'))]",
          "mode": "serial"
      },
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {
          },
          "variables": {
          },
          "resources": [
            {
              "type": "Microsoft.Network/virtualNetworks/virtualNetworkPeerings",
              "apiVersion": "2016-06-01",
              "location": "[resourceGroup().location]",
              "name": "[variables('peerings')[copyIndex()].name]",
              "properties": "[variables('peerings')[copyIndex()].properties]"
            }
          ],
          "outputs": {
          }
        }
      }
    }
]
```
Cette partie de notre modèle effectue plusieurs opérations. Pour commencer, la ressource réelle en cours de déploiement est un modèle inclus de type `Microsoft.Resources/deployments` intégrant son propre modèle qui procède au déploiement proprement dit de la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.

Nous garantissons l’unicité de l’élément `name` du modèle inclus en concaténant l’itération actuelle de l’élément `copyIndex()` avec le préfixe `vnp-`. 

L’élément `condition` indique que notre ressource doit être traitée lorsque la fonction `greater()` prend la valeur `true`. Ici, nous vérifions si le tableau de paramètres `virtualNetworkPeerings` est supérieur à zéro (`greater()`). Si tel est le cas, il prend la valeur `true`, et la `condition` est satisfaite. Dans le cas contraire, il prend la valeur `false`.

Ensuite, nous spécifions notre boucle `copy`. Il s’agit d’une boucle `serial`, ce qui signifie que la boucle est exécutée dans l’ordre, chaque ressource attendant que la dernière ressource ait été déployée. La propriété `count` spécifie le nombre d’itérations de la boucle. Ici, nous devrions théoriquement définir cette propriété sur la longueur du tableau `virtualNetworkPeerings`, car ce dernier contient les objets de paramètre qui spécifient la ressource que nous souhaitons déployer. Mais si nous effectuons cette opération, la validation échouera si le tableau est vide, car Resource Manager détectera que nous tentons d’accéder à des propriétés qui n’existent pas. Toutefois, nous pouvons contourner ce problème. Examinons les variables dont nous aurons besoin :

```json
  "variables": {
    "workaround": {
       "true": "[parameters('virtualNetworkPeerings')]",
       "false": [{
           "name": "workaround",
           "properties": {}
       }]
     },
     "peerings": "[variables('workaround')[string(greater(length(parameters('virtualNetworkPeerings')), 0))]]"
  },
```

Notre variable `workaround` inclut deux propriétés, `true` et `false`. La propriété `true` prend la valeur du tableau de paramètres `virtualNetworkPeerings`. La propriété `false` prend la valeur d’un objet vide incluant les propriétés nommées qui sont attendues par Resource Manager &mdash; notez que `false` correspond réellement à un tableau, tout comme notre paramètre `virtualNetworkPeerings`, ce qui entraînera la réussite de la validation. 

Notre variable `peerings` utilise notre variable `workaround` en vérifiant de nouveau si la longueur du tableau de paramètres `virtualNetworkPeerings` est supérieure à zéro. Si tel est le cas, l’élément `string` prend la valeur `true`, et la variable `workaround` prend la valeur du tableau de paramètres `virtualNetworkPeerings`. Dans le cas contraire, l’élément prend la valeur `false`, et la variable `workaround` prend la valeur de notre objet vide dans le premier élément du tableau.

À présent que nous avons contourné le problème de validation, nous pouvons nous contenter de spécifier le déploiement de la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` dans le modèle imbriqué en transmettant les éléments `name` et `properties` de notre tableau de paramètres `virtualNetworkPeerings`. Vous pouvez observer ceci dans l’élément `template` imbriqué dans l’élément `properties` de notre ressource.

## <a name="next-steps"></a>Étapes suivantes

* Cette technique est implémentée dans le [projet de blocs de construction de modèle](https://github.com/mspnp/template-building-blocks) et dans les [architectures de référence Azure](/azure/architecture/reference-architectures/). Vous pouvez utiliser ces derniers pour créer votre propre architecture ou déployer l’une de nos architectures de référence.

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-group-authoring-templates#resources
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings