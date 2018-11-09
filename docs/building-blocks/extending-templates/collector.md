---
title: Implémenter un transformateur et un collecteur de propriétés dans un modèle Azure Resource Manager
description: Décrit la procédure d’implémentation d’un transformateur et d’un collecteur de propriétés dans un modèle Azure Resource Manager
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: ad5b3a71f516ec12fee311e25c43f434f9f306ed
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251785"
---
# <a name="implement-a-property-transformer-and-collector-in-an-azure-resource-manager-template"></a>Implémenter un transformateur et un collecteur de propriétés dans un modèle Azure Resource Manager

Dans l’article [Utiliser un objet en tant que paramètre dans un modèle Azure Resource Manager][objects-as-parameters], vous avez découvert comment stocker des valeurs de propriété de ressource dans un objet et les appliquer à une ressource lors du déploiement. Bien que cette opération vous offre un moyen très utile de gérer vos paramètres, elle vous contraint toujours à mapper les propriétés de l’objet sur les propriétés de ressource chaque fois que vous utilisez cet objet dans votre modèle.

Pour contourner ce problème, vous pouvez implémenter un modèle de transformateur et de collecteur de propriétés qui itère votre tableau d’objets et le transforme en schéma JSON attendu par la ressource.

> [!IMPORTANT]
> Pour utiliser cette approche, vous devez disposer d’une connaissance approfondie des modèles et fonctions Resource Manager.

Examinons la façon dont nous pouvons implémenter un collecteur et un transformateur de propriétés à l’aide d’un exemple qui déploie un [Groupe de sécurité réseau (NSG)][nsg]. Le diagramme ci-après illustre la relation entre nos modèles et nos ressources au sein de ces modèles :

![Architecture du collecteur et du transformateur de propriétés](../_images/collector-transformer.png)

Notre **modèle d’appel** inclut deux ressources :
* un lien de modèle qui appelle notre **modèle de collecteur** ;
* la ressource NSG à déployer.

Notre **modèle de collecteur** inclut deux ressources :
* une ressource **ancre** ;
* un lien de modèle qui appelle le modèle de transformateur dans une boucle de copie.

Notre **modèle de transformateur** ne comporte qu’une seule ressource : un modèle vide avec une variable qui transforme notre code JSON `source` en schéma JSON attendu par notre ressource NSG dans le **modèle principal**.

## <a name="parameter-object"></a>Objet de paramètre

Nous allons utiliser notre objet de paramètre `securityRules` mentionné dans l’article décrivant [l’utilisation d’un objet en tant que paramètre][objects-as-parameters]. Notre **modèle de transformateur** transformera chaque objet du tableau `securityRules` en schéma JSON attendu par la ressource NSG dans notre **modèle d’appel**.

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

Commençons par examiner notre **modèle de transformateur**.

## <a name="transform-template"></a>Modèle de transformateur

Notre **modèle de transformateur** comporte deux paramètres qui sont transmis par le **modèle de collecteur** : 
* `source` est un objet qui reçoit l’un des objets de valeur de propriété du tableau de propriétés. Dans notre exemple, les différents objets du tableau `"securityRules"` sont transmis un par un.
* `state` est un tableau qui reçoit les résultats concaténés de toutes les transformations précédentes. Il s’agit de la collection de code JSON transformé.

Nos paramètres ressemblent à ceci :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "source": { "type": "object" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
  },
```

Notre modèle définit également une variable nommée `instance`. Elle effectue la transformation proprement dite de notre objet `source` en schéma JSON requis :

```json
  "variables": {
    "instance": [
      {
        "name": "[parameters('source').name]",
        "properties":{
            "description": "[parameters('source').description]",
            "protocol": "[parameters('source').protocol]",
            "sourcePortRange": "[parameters('source').sourcePortRange]",
            "destinationPortRange": "[parameters('source').destinationPortRange]",
            "sourceAddressPrefix": "[parameters('source').sourceAddressPrefix]",
            "destinationAddressPrefix": "[parameters('source').destinationAddressPrefix]",
            "access": "[parameters('source').access]",
            "priority": "[parameters('source').priority]",
            "direction": "[parameters('source').direction]"            
        }
      }
    ]

  },
```

Enfin, l’élément `output` de notre modèle concatène les transformations collectées de notre paramètre `state` avec la transformation actuelle effectuée par notre variable `instance` :

```json
    "resources": [],
    "outputs": {
    "collection": {
      "type": "array",
      "value": "[concat(parameters('state'), variables('instance'))]"
    }
```

À présent, examinons notre **modèle de collecteur** pour découvrir la façon dont il transmet nos valeurs de paramètre.

## <a name="collector-template"></a>Modèle de collecteur

Notre **modèle de collecteur** comporte trois paramètres :
* `source` est notre tableau d’objets de paramètre complet. Il est transmis par le **modèle d’appel**. Il porte le même nom que le paramètre `source` dans notre **modèle de transformateur**, mais présente une différence essentielle avec ce dernier que vous avez peut-être déjà remarquée : il s’agit du tableau complet, mais nous ne transmettons au **modèle de transformateur** qu’un seul élément de ce tableau à la fois.
* `transformTemplateUri` est l’URI de notre **modèle de transformateur**. Nous le définissons ici sous la forme d’un paramètre pour permettre la réutilisation du modèle.
* `state` est un tableau initialement vide que nous transmettons à notre **modèle de transformateur**. Il stocke la collection d’objets de paramètre transformés lorsque la boucle de copie est terminée.

Nos paramètres ressemblent à ceci :

```json
  "parameters": {
    "source": { "type": "array" },
    "transformTemplateUri": { "type": "string" },
    "state": {
      "type": "array",
      "defaultValue": [ ]
    }
``` 

Ensuite, nous définissons une variable nommée `count`. Sa valeur correspond à la longueur du tableau d’objets de paramètre `source` :

```json
  "variables": {
    "count": "[length(parameters('source'))]"
  },
```

Comme vous pouvez l’imaginer, nous l’utilisons pour le nombre d’itérations dans notre boucle de copie.

À présent, examinons nos ressources. Nous définissons deux ressources :
* `loop-0` est la ressource de base zéro pour notre boucle de copie.
* `loop-` est concaténé avec le résultat de la fonction `copyIndex(1)` afin de générer pour notre ressource un nom unique basé sur l’itération, en commençant par `1`.

Nos ressources ressemblent à ceci :

```json
  "resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "loop-0",
      "properties": {
        "mode": "Incremental",
        "parameters": { },
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": { },
          "variables": { },
          "resources": [ ],
          "outputs": {
            "collection": {
              "type": "array",
              "value": "[parameters('state')]"
            }
          }
        }
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2015-01-01",
      "name": "[concat('loop-', copyindex(1))]",
      "copy": {
        "name": "iterator",
        "count": "[variables('count')]",
        "mode": "serial"
      },
      "dependsOn": [
        "loop-0"
      ],
      "properties": {
        "mode": "Incremental",
        "templateLink": { "uri": "[parameters('transformTemplateUri')]" },
        "parameters": {
          "source": { "value": "[parameters('source')[copyindex()]]" },
          "state": { "value": "[reference(concat('loop-', copyindex())).outputs.collection.value]" }
        }
      }
    }
  ],
```

Examinons plus attentivement les paramètres que nous transmettons à notre **modèle de transformateur** dans le modèle imbriqué. Comme indiqué plus haut, notre paramètre `source` transmet l’objet actuel au tableau d’objets de paramètre `source`. Le paramètre `state` indique l’endroit où apparaît la collection, car il utilise la sortie de l’itération précédente de notre boucle de copie &mdash; notez que la fonction `reference()` utilise la fonction `copyIndex()` sans aucun paramètre pour référencer l’élément `name` de notre objet de modèle précédemment lié &mdash;, puis transmet cette sortie à l’itération actuelle.

Enfin, l’élément `output` de notre modèle renvoie l’élément `output` de la dernière itération de notre **modèle de transformateur** :

```json
  "outputs": {
    "result": {
      "type": "array",
      "value": "[reference(concat('loop-', variables('count'))).outputs.collection.value]"
    }
  }
```
Il peut sembler illogique de renvoyer l’élément `output` de la dernière itération de notre **modèle de transformateur** à notre **modèle d’appel**, puisqu’il s’avère que nous l’avons stocké dans notre paramètre `source`. Toutefois, n’oubliez pas qu’il s’agit de la dernière itération de notre **modèle de transformateur** qui stocke le tableau complet d’objets de propriété transformés, lequel correspond précisément à l’élément que nous souhaitons renvoyer.

Enfin, examinons la façon dont nous pouvons appeler le **modèle de collecteur** à partir de notre **modèle d’appel**.

## <a name="calling-template"></a>Modèle d’appel

Notre **modèle d’appel** définit un paramètre unique nommé `networkSecurityGroupsSettings` :

```json
...
"parameters": {
    "networkSecurityGroupsSettings": {
        "type": "object"
    }
```

Ensuite, notre modèle définit une seule variable nommée `collectorTemplateUri` :

```json
"variables": {
    "collectorTemplateUri": "[uri(deployment().properties.templateLink.uri, 'collector.template.json')]"
  }
```

Il s’agit évidemment de l’URI du **modèle de collecteur** qui sera utilisé par notre ressource de modèle lié :

```json
{
    "apiVersion": "2015-01-01",
    "name": "collector",
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "templateLink": {
            "uri": "[variables('collectorTemplateUri')]",
            "contentVersion": "1.0.0.0"
        },
        "parameters": {
            "source" : {"value": "[parameters('networkSecurityGroupsSettings').securityRules]"},
            "transformTemplateUri": { "value": "[uri(deployment().properties.templateLink.uri, 'transform.json')]"}
        }
    }
}
```

Nous transmettons deux paramètres au **modèle de collecteur** :
* `source` est notre tableau d’objets de propriété. Dans notre exemple, il s’agit de notre paramètre `networkSecurityGroupsSettings`.
* `transformTemplateUri` est la variable que nous venons de définir avec l’URI de notre **modèle de collecteur**.

Enfin, notre ressource `Microsoft.Network/networkSecurityGroups` attribue directement l’élément `output` de la ressource de modèle lié `collector` à sa propriété `securityRules` :

```json
    {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Network/networkSecurityGroups",
      "name": "networkSecurityGroup1",
      "location": "[resourceGroup().location]",
      "properties": {
        "securityRules": "[reference('collector').outputs.result.value]"
      }
    }
  ],
  "outputs": {
      "instance":{
          "type": "array",
          "value": "[reference('collector').outputs.result.value]"
      }

  }
```

## <a name="try-the-template"></a>Essayer le modèle

Un exemple de modèle est disponible sur [GitHub][github]. Pour déployer le modèle, clonez le référentiel et exécutez les commandes [Azure CLI][cli] suivantes :

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example4-collector
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example4-collector/deploy.json \
    --parameters deploy.parameters.json
```

<!-- links -->
[objects-as-parameters]: ./objects-as-parameters.md
[resource-manager-linked-template]: /azure/azure-resource-manager/resource-group-linked-templates
[resource-manager-variables]: /azure/azure-resource-manager/resource-group-template-functions-deployment
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
