---
title: Utiliser un objet en tant que paramètre dans un modèle Azure Resource Manager
description: Décrit comment étendre la fonctionnalité des modèles Azure Resource Manager pour utiliser des objets en tant que paramètres.
author: petertay
ms.date: 10/30/2018
ms.openlocfilehash: f0826d8ed1ce446d295ebdacc66d8b0bef0b0dec
ms.sourcegitcommit: 1f4cdb08fe73b1956e164ad692f792f9f635b409
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/08/2019
ms.locfileid: "54111204"
---
# <a name="use-an-object-as-a-parameter-in-an-azure-resource-manager-template"></a><span data-ttu-id="d2f13-103">Utiliser un objet en tant que paramètre dans un modèle Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="d2f13-103">Use an object as a parameter in an Azure Resource Manager template</span></span>

<span data-ttu-id="d2f13-104">Lorsque vous [créez des modèles Azure Resource Manager][azure-resource-manager-create-template], vous pouvez soit spécifier directement les valeurs de propriété de ressource dans le modèle, soit définir un paramètre et en indiquer les valeurs lors du déploiement.</span><span class="sxs-lookup"><span data-stu-id="d2f13-104">When you [author Azure Resource Manager templates][azure-resource-manager-create-template], you can either specify resource property values directly in the template or define a parameter and provide values during deployment.</span></span> <span data-ttu-id="d2f13-105">Vous avez la possibilité d’utiliser un paramètre pour chaque valeur de propriété dans le cas de petits déploiements, mais il existe une limite de 255 paramètres par déploiement.</span><span class="sxs-lookup"><span data-stu-id="d2f13-105">It's fine to use a parameter for each property value for small deployments, but there is a limit of 255 parameters per deployment.</span></span> <span data-ttu-id="d2f13-106">Une fois que vous devez prendre en charge des déploiements plus importants et plus complexes, vous pouvez vous trouver à court de paramètres.</span><span class="sxs-lookup"><span data-stu-id="d2f13-106">Once you get to larger and more complex deployments you may run out of parameters.</span></span>

<span data-ttu-id="d2f13-107">L’un des moyens de résoudre ce problème consiste à utiliser un objet en tant que paramètre au lieu d’une valeur.</span><span class="sxs-lookup"><span data-stu-id="d2f13-107">One way to solve this problem is to use an object as a parameter instead of a value.</span></span> <span data-ttu-id="d2f13-108">Pour effectuer cette opération, définissez le paramètre dans votre modèle, puis spécifiez un objet JSON au lieu d’une valeur unique au cours du déploiement.</span><span class="sxs-lookup"><span data-stu-id="d2f13-108">To do this, define the parameter in your template and specify a JSON object instead of a single value during deployment.</span></span> <span data-ttu-id="d2f13-109">Ensuite, référencez les sous-propriétés du paramètre en utilisant la [fonction `parameter()`][azure-resource-manager-functions] et l’opérateur point dans votre modèle.</span><span class="sxs-lookup"><span data-stu-id="d2f13-109">Then, reference the subproperties of the parameter using the [`parameter()` function][azure-resource-manager-functions] and dot operator in your template.</span></span>

<span data-ttu-id="d2f13-110">Examinons un exemple qui déploie une ressource de réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="d2f13-110">Let's take a look at an example that deploys a virtual network resource.</span></span> <span data-ttu-id="d2f13-111">Commençons par spécifier un paramètre `VNetSettings` dans notre modèle et à définir l’élément `type` sur `object` :</span><span class="sxs-lookup"><span data-stu-id="d2f13-111">First, let's specify a `VNetSettings` parameter in our template and set the `type` to `object`:</span></span>

```json
...
"parameters": {
    "VNetSettings":{"type":"object"}
},
```

<span data-ttu-id="d2f13-112">Ensuite, indiquons les valeurs de l’objet `VNetSettings` :</span><span class="sxs-lookup"><span data-stu-id="d2f13-112">Next, let's provide values for the `VNetSettings` object:</span></span>

> [!NOTE]
> <span data-ttu-id="d2f13-113">Pour découvrir comment spécifier les valeurs de paramètre lors du déploiement, consultez la section **parameters** de l’article [Comprendre la structure et la syntaxe des modèles Azure Resource Manager][azure-resource-manager-authoring-templates].</span><span class="sxs-lookup"><span data-stu-id="d2f13-113">To learn how to provide parameter values during deployment, see the **parameters** section of [understand the structure and syntax of Azure Resource Manager templates][azure-resource-manager-authoring-templates].</span></span>

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

<span data-ttu-id="d2f13-114">Comme vous pouvez le constater, notre paramètre unique spécifie en réalité trois sous-propriétés : `name`, `addressPrefixes` et `subnets`.</span><span class="sxs-lookup"><span data-stu-id="d2f13-114">As you can see, our single parameter actually specifies three subproperties: `name`, `addressPrefixes`, and `subnets`.</span></span> <span data-ttu-id="d2f13-115">Chacune de ces sous-propriétés indique soit une valeur, soit d’autres sous-propriétés.</span><span class="sxs-lookup"><span data-stu-id="d2f13-115">Each of these subproperties either specifies a value or other subproperties.</span></span> <span data-ttu-id="d2f13-116">En conséquence, notre paramètre spécifie à lui seul toutes les valeurs nécessaires au déploiement de notre réseau virtuel.</span><span class="sxs-lookup"><span data-stu-id="d2f13-116">The result is that our single parameter specifies all the values necessary to deploy our virtual network.</span></span>

<span data-ttu-id="d2f13-117">À présent, examinons le reste de notre modèle pour découvrir la façon dont l’objet `VNetSettings` est utilisé :</span><span class="sxs-lookup"><span data-stu-id="d2f13-117">Now let's have a look at the rest of our template to see how the `VNetSettings` object is used:</span></span>

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

<span data-ttu-id="d2f13-118">Les valeurs de notre objet `VNetSettings` sont appliquées aux propriétés requises par notre ressource de réseau virtuel à l’aide de la fonction `parameters()` avec l’indexeur de tableau `[]` et l’opérateur point.</span><span class="sxs-lookup"><span data-stu-id="d2f13-118">The values of our `VNetSettings` object are applied to the properties required by our virtual network resource using the `parameters()` function with both the `[]` array indexer and the dot operator.</span></span> <span data-ttu-id="d2f13-119">Cette approche fonctionne si vous souhaitez simplement appliquer de manière statique les valeurs de l’objet de paramètre à la ressource.</span><span class="sxs-lookup"><span data-stu-id="d2f13-119">This approach works if you just want to statically apply the values of the parameter object to the resource.</span></span> <span data-ttu-id="d2f13-120">En revanche, si vous voulez attribuer dynamiquement un tableau de valeurs de propriété lors du déploiement, vous pouvez utiliser une [boucle de copie][azure-resource-manager-create-multiple-instances].</span><span class="sxs-lookup"><span data-stu-id="d2f13-120">However, if you want to dynamically assign an array of property values during deployment you can use a [copy loop][azure-resource-manager-create-multiple-instances].</span></span> <span data-ttu-id="d2f13-121">Pour utiliser une boucle de copie, vous fournissez un tableau JSON de valeurs de propriété de ressource, et la boucle de copie applique alors dynamiquement ces valeurs aux propriétés de la ressource.</span><span class="sxs-lookup"><span data-stu-id="d2f13-121">To use a copy loop, you provide a JSON array of resource property values and the copy loop dynamically applies the values to the resource's properties.</span></span>

<span data-ttu-id="d2f13-122">Notez que l’utilisation de l’approche dynamique soulève un problème.</span><span class="sxs-lookup"><span data-stu-id="d2f13-122">There is one issue to be aware of if you use the dynamic approach.</span></span> <span data-ttu-id="d2f13-123">Pour illustrer ce dernier, examinons un tableau standard de valeurs de propriété.</span><span class="sxs-lookup"><span data-stu-id="d2f13-123">To demonstrate the issue, let's take a look at a typical array of property values.</span></span> <span data-ttu-id="d2f13-124">Dans cet exemple, les valeurs de nos propriétés sont stockées dans une variable.</span><span class="sxs-lookup"><span data-stu-id="d2f13-124">In this example the values for our properties are stored in a variable.</span></span> <span data-ttu-id="d2f13-125">Notez que nous disposons ici de deux tableaux &mdash; l’un nommé `firstProperty` et l’autre appelé `secondProperty`.</span><span class="sxs-lookup"><span data-stu-id="d2f13-125">Notice we have two arrays here&mdash;one named `firstProperty` and one named `secondProperty`.</span></span>

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

<span data-ttu-id="d2f13-126">À présent, observons la façon dont nous accédons aux propriétés dans la variable à l’aide d’une boucle de copie.</span><span class="sxs-lookup"><span data-stu-id="d2f13-126">Now let's take a look at the way we access the properties in the variable using a copy loop.</span></span>

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

<span data-ttu-id="d2f13-127">La fonction `copyIndex()` renvoie l’itération actuelle de la boucle de copie, que nous utilisons en tant qu’index dans chacun des deux tableaux simultanément.</span><span class="sxs-lookup"><span data-stu-id="d2f13-127">The `copyIndex()` function returns the current iteration of the copy loop, and we use that as an index into each of the two arrays simultaneously.</span></span>

<span data-ttu-id="d2f13-128">Cette approche fonctionne correctement lorsque les deux tableaux sont de même longueur.</span><span class="sxs-lookup"><span data-stu-id="d2f13-128">This works fine when the two arrays are the same length.</span></span> <span data-ttu-id="d2f13-129">Le problème apparaît si vous avez fait une erreur et que les deux tableaux ne présentent pas la même longueur &mdash; dans ce cas, la validation de votre modèle échouera lors du déploiement.</span><span class="sxs-lookup"><span data-stu-id="d2f13-129">The issue arises if you've made a mistake and the two arrays are different lengths&mdash;in this case your template will fail validation during deployment.</span></span> <span data-ttu-id="d2f13-130">Vous pouvez éviter ce problème en incluant toutes vos propriétés dans un même objet, car il est alors beaucoup plus facile de déterminer si une valeur est manquante.</span><span class="sxs-lookup"><span data-stu-id="d2f13-130">You can avoid this issue by including all your properties in a single object, because it is much easier to see when a value is missing.</span></span> <span data-ttu-id="d2f13-131">Par exemple, examinons un autre objet de paramètre dans lequel chaque élément du tableau `propertyObject` correspond à l’union des tableaux `firstProperty` et `secondProperty` précédemment utilisés.</span><span class="sxs-lookup"><span data-stu-id="d2f13-131">For example, let's take a look another parameter object in which each element of the `propertyObject` array is the union of the `firstProperty` and `secondProperty` arrays from earlier.</span></span>

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

<span data-ttu-id="d2f13-132">Vous pouvez constater que le troisième élément du tableau</span><span class="sxs-lookup"><span data-stu-id="d2f13-132">Notice the third element in the array?</span></span> <span data-ttu-id="d2f13-133">est dépourvu de la propriété `number`. Cette méthode d’écriture des valeurs de paramètre vous permet donc de remarquer beaucoup plus facilement que vous avez oublié cette propriété.</span><span class="sxs-lookup"><span data-stu-id="d2f13-133">It's missing the `number` property, but it's much easier to notice that you've missed it when you're authoring the parameter values this way.</span></span>

## <a name="using-a-property-object-in-a-copy-loop"></a><span data-ttu-id="d2f13-134">Utilisation d’un objet de propriété dans une boucle de copie</span><span class="sxs-lookup"><span data-stu-id="d2f13-134">Using a property object in a copy loop</span></span>

<span data-ttu-id="d2f13-135">Cette approche se révèle encore plus utile lorsque vous l’utilisez conjointement avec la [boucle de copie en série][azure-resource-manager-create-multiple], notamment pour le déploiement de ressources enfants.</span><span class="sxs-lookup"><span data-stu-id="d2f13-135">This approach becomes even more useful when combined with the [serial copy loop][azure-resource-manager-create-multiple], particularly for deploying child resources.</span></span>

<span data-ttu-id="d2f13-136">Pour illustrer ce point, examinons un modèle qui déploie un [Groupe de sécurité réseau (NSG)][nsg] avec deux règles de sécurité.</span><span class="sxs-lookup"><span data-stu-id="d2f13-136">To demonstrate this, let's look at a template that deploys a [network security group (NSG)][nsg] with two security rules.</span></span>

<span data-ttu-id="d2f13-137">Commençons par étudier nos paramètres.</span><span class="sxs-lookup"><span data-stu-id="d2f13-137">First, let's take a look at our parameters.</span></span> <span data-ttu-id="d2f13-138">Lorsque nous examinerons notre modèle, nous constaterons que nous avons défini un seul paramètre nommé `networkSecurityGroupsSettings` qui inclut un tableau appelé `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="d2f13-138">When we look at our template we'll see that we've defined one parameter named `networkSecurityGroupsSettings` that includes an array named `securityRules`.</span></span> <span data-ttu-id="d2f13-139">Ce tableau contient deux objets JSON qui spécifient un certain nombre de paramètres pour une règle de sécurité.</span><span class="sxs-lookup"><span data-stu-id="d2f13-139">This array contains two JSON objects that specify a number of settings for a security rule.</span></span>

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

<span data-ttu-id="d2f13-140">À présent, examinons notre modèle.</span><span class="sxs-lookup"><span data-stu-id="d2f13-140">Now let's take a look at our template.</span></span> <span data-ttu-id="d2f13-141">Notre première ressource nommée `NSG1` déploie le NSG.</span><span class="sxs-lookup"><span data-stu-id="d2f13-141">Our first resource named `NSG1` deploys the NSG.</span></span> <span data-ttu-id="d2f13-142">Notre deuxième ressource, nommée `loop-0`, accomplit deux fonctions : en premier lieu, l’élément `dependsOn` indique qu’elle dépend du NSG, de sorte que son déploiement ne commence pas avant que la ressource `NSG1` ait rempli son rôle, et en second lieu, elle constitue la première itération de la boucle séquentielle.</span><span class="sxs-lookup"><span data-stu-id="d2f13-142">Our second resource named `loop-0` performs two functions: first, it `dependsOn` the NSG so its deployment doesn't begin until `NSG1` is completed, and it is the first iteration of the sequential loop.</span></span> <span data-ttu-id="d2f13-143">Notre troisième ressource est un modèle imbriqué qui déploie nos règles de sécurité à l’aide d’un objet pour ses valeurs de paramètre comme dans l’exemple précédent.</span><span class="sxs-lookup"><span data-stu-id="d2f13-143">Our third resource is a nested template that deploys our security rules using an object for its parameter values as in the last example.</span></span>

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

<span data-ttu-id="d2f13-144">Examinons de plus près la façon dont nous spécifions nos valeurs de propriété dans la ressource enfant `securityRules`.</span><span class="sxs-lookup"><span data-stu-id="d2f13-144">Let's take a closer look at how we specify our property values in the `securityRules` child resource.</span></span> <span data-ttu-id="d2f13-145">Toutes nos propriétés sont référencées à l’aide de la fonction `parameter()`, puis nous utilisons l’opérateur point pour référencer notre tableau `securityRules`, indexé par la valeur actuelle de l’itération.</span><span class="sxs-lookup"><span data-stu-id="d2f13-145">All of our properties are referenced using the `parameter()` function, and then we use the dot operator to reference our `securityRules` array, indexed by the current value of the iteration.</span></span> <span data-ttu-id="d2f13-146">Enfin, nous utilisons un autre opérateur point pour référencer le nom de l’objet.</span><span class="sxs-lookup"><span data-stu-id="d2f13-146">Finally, we use another dot operator to reference the name of the object.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="d2f13-147">Essayer le modèle</span><span class="sxs-lookup"><span data-stu-id="d2f13-147">Try the template</span></span>

<span data-ttu-id="d2f13-148">Un exemple de modèle est disponible sur [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="d2f13-148">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="d2f13-149">Pour déployer le modèle, clonez le référentiel et exécutez les commandes [Azure CLI][cli] suivantes :</span><span class="sxs-lookup"><span data-stu-id="d2f13-149">To deploy the template, clone the repo and run the following [Azure CLI][cli] commands:</span></span>

```bash
git clone https://github.com/mspnp/template-examples.git
cd template-examples/example3-object-param
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example3-object-param/deploy.json \
    --parameters deploy.parameters.json
```

## <a name="next-steps"></a><span data-ttu-id="d2f13-150">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="d2f13-150">Next steps</span></span>

- <span data-ttu-id="d2f13-151">Découvrez comment créer un modèle qui effectue une itération dans un tableau d’objets et le transforme en schéma JSON.</span><span class="sxs-lookup"><span data-stu-id="d2f13-151">Learn how to create a template that iterates through an object array and transforms it into a JSON schema.</span></span> <span data-ttu-id="d2f13-152">Voir [Implémenter un transformateur et un collecteur de propriétés dans un modèle Azure Resource Manager](./collector.md)</span><span class="sxs-lookup"><span data-stu-id="d2f13-152">See [Implement a property transformer and collector in an Azure Resource Manager template](./collector.md)</span></span>

<!-- links -->

[azure-resource-manager-authoring-templates]: /azure/azure-resource-manager/resource-group-authoring-templates
[azure-resource-manager-create-template]: /azure/azure-resource-manager/resource-manager-create-first-template
[azure-resource-manager-create-multiple-instances]: /azure/azure-resource-manager/resource-group-create-multiple
[azure-resource-manager-functions]: /azure/azure-resource-manager/resource-group-template-functions-resource
[nsg]: /azure/virtual-network/virtual-networks-nsg
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
