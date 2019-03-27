---
title: Déployer une ressource de manière conditionnelle dans un modèle Azure Resource Manager
description: Explique comment étendre les fonctionnalités des modèles Azure Resource Manager pour déployer une ressource de manière conditionnelle en fonction de la valeur d’un paramètre.
author: petertay
ms.date: 10/30/2018
ms.topic: article
ms.service: architecture-center
ms.subservice: reference-architecture
ms.openlocfilehash: f3d22c6437cdabcd93a781ecf7c99db5a570d7cf
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 03/20/2019
ms.locfileid: "58243800"
---
# <a name="conditionally-deploy-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="62aad-103">Déployer une ressource de manière conditionnelle dans un modèle Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="62aad-103">Conditionally deploy a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="62aad-104">Certains scénarios nécessitent que vous conceviez votre modèle pour le déploiement d’une ressource reposant sur une condition spécifique, telle que la présence ou l’absence d’une valeur de paramètre.</span><span class="sxs-lookup"><span data-stu-id="62aad-104">There are some scenarios in which you need to design your template to deploy a resource based on a condition, such as whether or not a parameter value is present.</span></span> <span data-ttu-id="62aad-105">Par exemple, votre modèle peut déployer un réseau virtuel et inclure des paramètres pour spécifier d’autres réseaux virtuels à des fins d’homologation.</span><span class="sxs-lookup"><span data-stu-id="62aad-105">For example, your template may deploy a virtual network and include parameters to specify other virtual networks for peering.</span></span> <span data-ttu-id="62aad-106">Si vous n’avez spécifié aucune valeur de paramètre pour l’homologation, vous ne souhaitez pas que Resource Manager déploie la ressource d’homologation.</span><span class="sxs-lookup"><span data-stu-id="62aad-106">If you've not specified any parameter values for peering, you don't want Resource Manager to deploy the peering resource.</span></span>

<span data-ttu-id="62aad-107">Pour effectuer cette opération, utilisez l’[élément condition][azure-resource-manager-condition] dans la ressource afin de tester la longueur de votre tableau de paramètres.</span><span class="sxs-lookup"><span data-stu-id="62aad-107">To accomplish this, use the [condition element][azure-resource-manager-condition] in the resource to test the length of your parameter array.</span></span> <span data-ttu-id="62aad-108">Si cette longueur est égale à zéro, renvoyez la valeur `false` pour empêcher le déploiement ; pour toutes les longueurs supérieures à zéro, renvoyez la valeur `true` afin d’autoriser le déploiement.</span><span class="sxs-lookup"><span data-stu-id="62aad-108">If the length is zero, return `false` to prevent deployment, but for all values greater than zero return `true` to allow deployment.</span></span>

## <a name="example-template"></a><span data-ttu-id="62aad-109">Exemple de modèle</span><span class="sxs-lookup"><span data-stu-id="62aad-109">Example template</span></span>

<span data-ttu-id="62aad-110">Examinons un exemple de modèle illustrant cette approche.</span><span class="sxs-lookup"><span data-stu-id="62aad-110">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="62aad-111">Notre modèle utilise l’[élément condition][azure-resource-manager-condition] pour contrôler le déploiement de la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="62aad-111">Our template uses the [condition element][azure-resource-manager-condition] to control deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource.</span></span> <span data-ttu-id="62aad-112">Cette ressource crée une homologation entre deux réseaux virtuels Azure dans la même région.</span><span class="sxs-lookup"><span data-stu-id="62aad-112">This resource creates a peering between two Azure Virtual Networks in the same region.</span></span>

<span data-ttu-id="62aad-113">Examinons chaque section du modèle.</span><span class="sxs-lookup"><span data-stu-id="62aad-113">Let's take a look at each section of the template.</span></span>

<span data-ttu-id="62aad-114">L’élément `parameters` définit un paramètre unique nommé `virtualNetworkPeerings` :</span><span class="sxs-lookup"><span data-stu-id="62aad-114">The `parameters` element defines a single parameter named `virtualNetworkPeerings`:</span></span>

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

<span data-ttu-id="62aad-115">Notre paramètre `virtualNetworkPeerings` est un `array` et présente le schéma suivant :</span><span class="sxs-lookup"><span data-stu-id="62aad-115">Our `virtualNetworkPeerings` parameter is an `array` and has the following schema:</span></span>

```json
"virtualNetworkPeerings": [
    {
      "name": "firstVNet/peering1",
      "properties": {
          "remoteVirtualNetwork": {
              "id": "[resourceId('Microsoft.Network/virtualNetworks','secondVNet')]"
          },
          "allowForwardedTraffic": true,
          "allowGatewayTransit": true,
          "useRemoteGateways": false
      }
    }
]
```

<span data-ttu-id="62aad-116">Les propriétés de notre paramètre spécifient les [paramètres associés à l’homologation de réseaux virtuels][vnet-peering-resource-schema].</span><span class="sxs-lookup"><span data-stu-id="62aad-116">The properties in our parameter specify the [settings related to peering virtual networks][vnet-peering-resource-schema].</span></span> <span data-ttu-id="62aad-117">Nous fournissons les valeurs de ces propriétés lorsque nous spécifions la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` dans la section `resources` :</span><span class="sxs-lookup"><span data-stu-id="62aad-117">We'll provide the values for these properties when we specify the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the `resources` section:</span></span>

```json
"resources": [
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2017-05-10",
      "name": "[concat('vnp-', copyIndex())]",
      "condition": "[greater(length(parameters('virtualNetworkPeerings')), 0)]",
      "dependsOn": [
        "firstVNet", "secondVNet"
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

<span data-ttu-id="62aad-118">Cette partie de notre modèle effectue plusieurs opérations.</span><span class="sxs-lookup"><span data-stu-id="62aad-118">There are a couple of things going on in this part of our template.</span></span> <span data-ttu-id="62aad-119">Pour commencer, la ressource réelle en cours de déploiement est un modèle inclus de type `Microsoft.Resources/deployments` intégrant son propre modèle qui procède au déploiement proprement dit de la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="62aad-119">First, the actual resource being deployed is an inline template of type `Microsoft.Resources/deployments` that includes its own template that actually deploys the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings`.</span></span>

<span data-ttu-id="62aad-120">Nous garantissons l’unicité de l’élément `name` du modèle inclus en concaténant l’itération actuelle de l’élément `copyIndex()` avec le préfixe `vnp-`.</span><span class="sxs-lookup"><span data-stu-id="62aad-120">Our `name` for the inline template is made unique by concatenating the current iteration of the `copyIndex()` with the prefix `vnp-`.</span></span>

<span data-ttu-id="62aad-121">L’élément `condition` indique que notre ressource doit être traitée lorsque la fonction `greater()` prend la valeur `true`.</span><span class="sxs-lookup"><span data-stu-id="62aad-121">The `condition` element specifies that our resource should be processed when the `greater()` function evaluates to `true`.</span></span> <span data-ttu-id="62aad-122">Ici, nous vérifions si le tableau de paramètres `virtualNetworkPeerings` est supérieur à zéro (`greater()`).</span><span class="sxs-lookup"><span data-stu-id="62aad-122">Here, we're testing if the `virtualNetworkPeerings` parameter array is `greater()` than zero.</span></span> <span data-ttu-id="62aad-123">Si tel est le cas, il prend la valeur `true`, et la `condition` est satisfaite.</span><span class="sxs-lookup"><span data-stu-id="62aad-123">If it is, it evaluates to `true` and the `condition` is satisfied.</span></span> <span data-ttu-id="62aad-124">Dans le cas contraire, il prend la valeur `false`.</span><span class="sxs-lookup"><span data-stu-id="62aad-124">Otherwise, it's `false`.</span></span>

<span data-ttu-id="62aad-125">Ensuite, nous spécifions notre boucle `copy`.</span><span class="sxs-lookup"><span data-stu-id="62aad-125">Next, we specify our `copy` loop.</span></span> <span data-ttu-id="62aad-126">Il s’agit d’une boucle `serial`, ce qui signifie que la boucle est exécutée dans l’ordre, chaque ressource attendant que la dernière ressource ait été déployée.</span><span class="sxs-lookup"><span data-stu-id="62aad-126">It's a `serial` loop that means the loop is done in sequence, with each resource waiting until the last resource has been deployed.</span></span> <span data-ttu-id="62aad-127">La propriété `count` spécifie le nombre d’itérations de la boucle.</span><span class="sxs-lookup"><span data-stu-id="62aad-127">The `count` property specifies the number of times the loop iterates.</span></span> <span data-ttu-id="62aad-128">Ici, nous devrions théoriquement définir cette propriété sur la longueur du tableau `virtualNetworkPeerings`, car ce dernier contient les objets de paramètre qui spécifient la ressource que nous souhaitons déployer.</span><span class="sxs-lookup"><span data-stu-id="62aad-128">Here, normally we'd set it to the length of the `virtualNetworkPeerings` array because it contains the parameter objects specifying the resource we want to deploy.</span></span> <span data-ttu-id="62aad-129">Mais si nous effectuons cette opération, la validation échouera si le tableau est vide, car Resource Manager détectera que nous tentons d’accéder à des propriétés qui n’existent pas.</span><span class="sxs-lookup"><span data-stu-id="62aad-129">However, if we do that, validation will fail if the array is empty because Resource Manager notices that we are attempting to access properties that do not exist.</span></span> <span data-ttu-id="62aad-130">Toutefois, nous pouvons contourner ce problème.</span><span class="sxs-lookup"><span data-stu-id="62aad-130">We can work around this, however.</span></span> <span data-ttu-id="62aad-131">Examinons les variables dont nous aurons besoin :</span><span class="sxs-lookup"><span data-stu-id="62aad-131">Let's take a look at the variables we'll need:</span></span>

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

<span data-ttu-id="62aad-132">Notre variable `workaround` inclut deux propriétés, `true` et `false`.</span><span class="sxs-lookup"><span data-stu-id="62aad-132">Our `workaround` variable includes two properties, one named `true` and one named `false`.</span></span> <span data-ttu-id="62aad-133">La propriété `true` prend la valeur du tableau de paramètres `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="62aad-133">The `true` property evaluates to the value of the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="62aad-134">La propriété `false` prend la valeur d’un objet vide incluant les propriétés nommées qui sont attendues par Resource Manager &mdash; notez que `false` correspond réellement à un tableau, tout comme notre paramètre `virtualNetworkPeerings`, ce qui entraîne la réussite de la validation.</span><span class="sxs-lookup"><span data-stu-id="62aad-134">The `false` property evaluates to an empty object including the named properties that Resource Manager expects to see &mdash; note that `false` is actually an array, just as our `virtualNetworkPeerings` parameter is, which will satisfy validation.</span></span>

<span data-ttu-id="62aad-135">Notre variable `peerings` utilise notre variable `workaround` en vérifiant de nouveau si la longueur du tableau de paramètres `virtualNetworkPeerings` est supérieure à zéro.</span><span class="sxs-lookup"><span data-stu-id="62aad-135">Our `peerings` variable uses our `workaround` variable by once again testing if the length of the `virtualNetworkPeerings` parameter array is greater than zero.</span></span> <span data-ttu-id="62aad-136">Si tel est le cas, l’élément `string` prend la valeur `true`, et la variable `workaround` prend la valeur du tableau de paramètres `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="62aad-136">If it is, the `string` evaluates to `true` and the `workaround` variable evaluates to the `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="62aad-137">Dans le cas contraire, l’élément prend la valeur `false`, et la variable `workaround` prend la valeur de notre objet vide dans le premier élément du tableau.</span><span class="sxs-lookup"><span data-stu-id="62aad-137">Otherwise, it evaluates to `false` and the `workaround` variable evaluates to our empty object in the first element of the array.</span></span>

<span data-ttu-id="62aad-138">À présent que nous avons contourné le problème de validation, nous pouvons nous contenter de spécifier le déploiement de la ressource `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` dans le modèle imbriqué en transmettant les éléments `name` et `properties` de notre tableau de paramètres `virtualNetworkPeerings`.</span><span class="sxs-lookup"><span data-stu-id="62aad-138">Now that we've worked around the validation issue, we can simply specify the deployment of the `Microsoft.Network/virtualNetworks/virtualNetworkPeerings` resource in the nested template, passing the `name` and `properties` from our `virtualNetworkPeerings` parameter array.</span></span> <span data-ttu-id="62aad-139">Vous pouvez observer ceci dans l’élément `template` imbriqué dans l’élément `properties` de notre ressource.</span><span class="sxs-lookup"><span data-stu-id="62aad-139">You can see this in the `template` element nested in the `properties` element of our resource.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="62aad-140">Essayer le modèle</span><span class="sxs-lookup"><span data-stu-id="62aad-140">Try the template</span></span>

<span data-ttu-id="62aad-141">Un exemple de modèle est disponible sur [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="62aad-141">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="62aad-142">Pour déployer le modèle, exécutez les commandes [Azure CLI][cli] suivantes :</span><span class="sxs-lookup"><span data-stu-id="62aad-142">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example2-conditional/deploy.json
```

## <a name="next-steps"></a><span data-ttu-id="62aad-143">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="62aad-143">Next steps</span></span>

* <span data-ttu-id="62aad-144">Utilisez des objets au lieu de valeurs scalaires comme paramètres de modèle.</span><span class="sxs-lookup"><span data-stu-id="62aad-144">Use objects instead of scalar values as template parameters.</span></span> <span data-ttu-id="62aad-145">Consultez [Utiliser un objet en tant que paramètre dans un modèle Azure Resource Manager](./objects-as-parameters.md)</span><span class="sxs-lookup"><span data-stu-id="62aad-145">See [Use an object as a parameter in an Azure Resource Manager template](./objects-as-parameters.md)</span></span>

<!-- links -->
[azure-resource-manager-condition]: /azure/azure-resource-manager/resource-manager-templates-resources#condition
[azure-resource-manager-variable]: /azure/azure-resource-manager/resource-group-authoring-templates#variables
[vnet-peering-resource-schema]: /azure/templates/microsoft.network/virtualnetworks/virtualnetworkpeerings
[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples
