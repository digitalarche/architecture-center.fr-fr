---
title: Utiliser un objet en tant que paramètre dans un modèle Azure Resource Manager
description: Explique comment étendre les fonctionnalités des modèles Azure Resource Manager afin d’utiliser des objets en tant que paramètres
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: c1955823b3474efa0abea1d9634add5f13d02eda
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251887"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a>Utiliser un objet en tant que paramètre dans un modèle Azure Resource Manager

Lorsque vous [créez des modèles Azure Resource Manager][azure-resource-manager-create-template], vous pouvez soit spécifier directement les valeurs de propriété de ressource dans le modèle, soit définir un paramètre et en indiquer les valeurs lors du déploiement. Vous avez la possibilité d’utiliser un paramètre pour chaque valeur de propriété dans le cas de petits déploiements, mais il existe une limite de 255 paramètres par déploiement. Une fois que vous devez prendre en charge des déploiements plus importants et plus complexes, vous pouvez vous trouver à court de paramètres.

L’un des moyens de résoudre ce problème consiste à utiliser un objet en tant que paramètre au lieu d’une valeur. Pour effectuer cette opération, définissez le paramètre dans votre modèle, puis spécifiez un objet JSON au lieu d’une valeur unique au cours du déploiement. Ensuite, référencez les sous-propriétés du paramètre en utilisant la [fonction `parameter()`][azure-resource-manager-functions] et l’opérateur point dans votre modèle.

Examinons un exemple qui déploie une ressource de réseau virtuel. Commençons par spécifier un paramètre `VNetSettings` dans notre modèle et à définir l’élément `type` sur `object` :

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```
Ensuite, indiquons les valeurs de l’objet `VNetSettings` :

> [!NOTE]
> Pour découvrir comment spécifier les valeurs de paramètre lors du déploiement, consultez la section **parameters** de l’article [Comprendre la structure et la syntaxe des modèles Azure Resource Manager][azure-resource-manager-authoring-templates]. 

```json
"parameters":{
    "VNetSettings":{
        "value":{
            "name":"VNet1",
            "addressPrefixes": [
                {
                    "name": "firstPrefix",
                    "addressPrefix": "10.0.0.0/22"
                }
            ],
            "subnets":[
                {
                    "name": "firstSubnet",
                    "addressPrefix": "10.0.0.0/24"
                },
                {
                    "name":"secondSubnet",
                    "addressPrefix":"10.0.1.0/24"
                }
            ]
        }
    }
}
```

Comme vous pouvez le constater, notre paramètre unique spécifie en réalité trois sous-propriétés : `name`, `addressPrefixes` et `subnets`. Chacune de ces sous-propriétés indique soit une valeur, soit d’autres sous-propriétés. En conséquence, notre paramètre spécifie à lui seul toutes les valeurs nécessaires au déploiement de notre réseau virtuel.

À présent, examinons le reste de notre modèle pour découvrir la façon dont l’objet `VNetSettings` est utilisé :

```json
...
"resources": [
    {
        "apiVersion": "2015-06-15",
        "type": "Microsoft.Network/virtualNetworks",
        "name": "[parameters('VNetSettings').name]",
        "location":"[resourceGroup().location]",
        "properties": {
          "addressSpace":{
              "addressPrefixes": [
                "[parameters('VNetSettings').addressPrefixes[0].addressPrefix]"
              ]
          },
          "subnets":[
              {
                  "name":"[parameters('VNetSettings').subnets[0].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[0].addressPrefix]"
                  }
              },
              {
                  "name":"[parameters('VNetSettings').subnets[1].name]",
                  "properties": {
                      "addressPrefix": "[parameters('VNetSettings').subnets[1].addressPrefix]"
                  }
              }
          ]
        }
    }
  ]
```
Les valeurs de notre objet `VNetSettings` sont appliquées aux propriétés requises par notre ressource de réseau virtuel à l’aide de la fonction `parameters()` avec l’indexeur de tableau `[]` et l’opérateur point. Cette approche fonctionne si vous souhaitez simplement appliquer de manière statique les valeurs de l’objet de paramètre à la ressource. En revanche, si vous voulez attribuer dynamiquement un tableau de valeurs de propriété lors du déploiement, vous pouvez utiliser une [boucle de copie][azure-resource-manager-create-multiple-instances]. Pour utiliser une boucle de copie, vous fournissez un tableau JSON de valeurs de propriété de ressource, et la boucle de copie applique alors dynamiquement ces valeurs aux propriétés de la ressource. 

Notez que l’utilisation de l’approche dynamique soulève un problème. Pour illustrer ce dernier, examinons un tableau standard de valeurs de propriété. Dans cet exemple, les valeurs de nos propriétés sont stockées dans une variable. Notez que nous disposons ici de deux tableaux &mdash; l’un nommé `firstProperty` et l’autre appelé `secondProperty`. 

```json
"variables": {
    "firstProperty": [
        {
            "name": "A",
            "type": "typeA"
        },
        {
            "name": "B",
            "type": "typeB"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ],
    "secondProperty": [
        "one","two", "three"
    ]
}
```

À présent, observons la façon dont nous accédons aux propriétés dans la variable à l’aide d’une boucle de copie.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    ...
    "copy": {
        "name": "copyLoop1",
        "count": "[length(variables('firstProperty'))]"
    },
    ...
    "properties": {
        "name": { "value": "[variables('firstProperty')[copyIndex()].name]" },
        "type": { "value": "[variables('firstProperty')[copyIndex()].type]" },
        "number": { "value": "[variables('secondProperty')[copyIndex()]]" }
    }
}
```

La fonction `copyIndex()` renvoie l’itération actuelle de la boucle de copie, que nous utilisons en tant qu’index dans chacun des deux tableaux simultanément.

Cette approche fonctionne correctement lorsque les deux tableaux sont de même longueur. Le problème apparaît si vous avez fait une erreur et que les deux tableaux ne présentent pas la même longueur &mdash; dans ce cas, la validation de votre modèle échouera lors du déploiement. Vous pouvez éviter ce problème en incluant toutes vos propriétés dans un même objet, car il est alors beaucoup plus facile de déterminer si une valeur est manquante. Par exemple, examinons un autre objet de paramètre dans lequel chaque élément du tableau `propertyObject` correspond à l’union des tableaux `firstProperty` et `secondProperty` précédemment utilisés.

```json
"variables": {
    "propertyObject": [
        {
            "name": "A",
            "type": "typeA",
            "number": "one"
        },
        {
            "name": "B",
            "type": "typeB",
            "number": "two"
        },
        {
            "name": "C",
            "type": "typeC"
        }
    ]
}
```

Vous pouvez constater que le troisième élément du tableau est dépourvu de la propriété `number`. Cette méthode d’écriture des valeurs de paramètre vous permet donc de remarquer beaucoup plus facilement que vous avez oublié cette propriété.

## <a name="using-a-property-object-in-a-copy-loop"></a>Utilisation d’un objet de propriété dans une boucle de copie

Cette approche se révèle encore plus utile lorsque vous l’utilisez conjointement avec la [boucle de copie en série][azure-resource-manager-create-multiple], notamment pour le déploiement de ressources enfants. 

Pour illustrer ce point, examinons un modèle qui déploie un [Groupe de sécurité réseau (NSG)][nsg] avec deux règles de sécurité. 

Commençons par étudier nos paramètres. Lorsque nous examinerons notre modèle, nous constaterons que nous avons défini un seul paramètre nommé `networkSecurityGroupsSettings` qui inclut un tableau appelé `securityRules`. Ce tableau contient deux objets JSON qui spécifient un certain nombre de paramètres pour une règle de sécurité.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters":{ 
      "networkSecurityGroupsSettings": {
      "value": {
          "securityRules": [
            {
              "name": "RDPAllow",
              "description": "allow RDP connections",
              "direction": "Inbound",
              "priority": 100,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.0.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "3389",
              "access": "Allow",
              "protocol": "Tcp"
            },
            {
              "name": "HTTPAllow",
              "description": "allow HTTP connections",
              "direction": "Inbound",
              "priority": 200,
              "sourceAddressPrefix": "*",
              "destinationAddressPrefix": "10.0.1.0/24",
              "sourcePortRange": "*",
              "destinationPortRange": "80",
              "access": "Allow",
              "protocol": "Tcp"
            }
          ]
        }
      }
    }
  }
```

À présent, examinons notre modèle. Notre première ressource nommée `NSG1` déploie le NSG. Notre deuxième ressource, nommée `loop-0`, accomplit deux fonctions : en premier lieu, l’élément `dependsOn` indique qu’elle dépend du NSG, de sorte que son déploiement ne commence pas avant que la ressource `NSG1` ait rempli son rôle, et en second lieu, elle constitue la première itération de la boucle séquentielle. Notre troisième ressource est un modèle imbriqué qui déploie nos règles de sécurité à l’aide d’un objet pour ses valeurs de paramètre comme dans l’exemple précédent.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "networkSecurityGroupsSettings": {"type":"object"}
  },
  "variables": {},
  "resources": [
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "NSG1",
      "location":"[resourceGroup().location]",
      "properties": {
          "securityRules":[]
      }
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "loop-0",
        "dependsOn": [
            "NSG1"
        ],
        "properties": {
            "mode":"Incremental",
            "parameters":{},
            "template": {
                "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
                "contentVersion": "1.0.0.0",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            }
        }       
    },
    {
        "apiVersion": "2015-01-01",
        "type": "Microsoft.Resources/deployments",
        "name": "[concat('loop-', copyIndex(1))]",
        "dependsOn": [
          "[concat('loop-', copyIndex())]"
        ],
        "copy": {
          "name": "iterator",
          "count": "[length(parameters('networkSecurityGroupsSettings').securityRules)]"
        },
        "properties": {
          "mode": "Incremental",
          "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
           "parameters": {},
            "variables": {},
            "resources": [
                {
                    "name": "[concat('NSG1/' , parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].name)]",
                    "type": "Microsoft.Network/networkSecurityGroups/securityRules",
                    "apiVersion": "2016-09-01",
                    "location":"[resourceGroup().location]",
                    "properties":{
                        "description": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].description]",
                        "priority":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].priority]",
                        "protocol":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].protocol]",
                        "sourcePortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourcePortRange]",
                        "destinationPortRange": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationPortRange]",
                        "sourceAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].sourceAddressPrefix]",
                        "destinationAddressPrefix": "[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].destinationAddressPrefix]",
                        "access":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].access]",
                        "direction":"[parameters('networkSecurityGroupsSettings').securityRules[copyIndex()].direction]"
                        }
                  }
            ],
            "outputs": {}
          }
        }
    }
  ],          
  "outputs": {}
}
```

Examinons de plus près la façon dont nous spécifions nos valeurs de propriété dans la ressource enfant `securityRules`. Toutes nos propriétés sont référencées à l’aide de la fonction `parameter()`, puis nous utilisons l’opérateur point pour référencer notre tableau `securityRules`, indexé par la valeur actuelle de l’itération. Enfin, nous utilisons un autre opérateur point pour référencer le nom de l’objet. 

## <a name="try-the-template"></a>Essayer le modèle

Un exemple de modèle est disponible sur [GitHub][github]. Pour déployer le modèle, clonez le référentiel et exécutez les commandes [Azure CLI][cli] suivantes :

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment créer un modèle qui effectue une itération dans un tableau d’objets et le transforme en schéma JSON. Voir [Implémenter un transformateur et un collecteur de propriétés dans un modèle Azure Resource Manager](./collector.md)


<!-- links -->
[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
