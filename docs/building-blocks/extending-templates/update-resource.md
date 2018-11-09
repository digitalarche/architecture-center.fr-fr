---
title: Mettre à jour une ressource dans un modèle Azure Resource Manager
description: Explique comment étendre les fonctionnalités des modèles Azure Resource Manager afin de mettre à jour une ressource
author: petertay
ms.date: 10/31/2018
ms.openlocfilehash: dc97534e658c9728ac617b4e52031e2553600458
ms.sourcegitcommit: e9eb2b895037da0633ef3ccebdea2fcce047620f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2018
ms.locfileid: "50251819"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="2d1b4-103">Mettre à jour une ressource dans un modèle Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="2d1b4-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="2d1b4-104">Il existe certains scénarios dans lesquels vous devez mettre à jour une ressource pendant un déploiement.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="2d1b4-105">Vous pouvez rencontrer ce scénario lorsque vous ne pouvez pas spécifier toutes les propriétés d’une ressource jusqu’à ce que les autres ressources dépendantes aient été créées.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="2d1b4-106">Par exemple, si vous créez un pool principal pour un équilibrage de charge, vous pouvez mettre à jour les interfaces réseau (NIC) sur vos machines virtuelles pour les inclure dans le pool principal.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="2d1b4-107">Alors que Resource Manager prend en charge la mise à jour des ressources lors du déploiement, vous devez concevoir votre modèle correctement pour éviter les erreurs et garantir que le déploiement est traité comme une mise à jour.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="2d1b4-108">Vous devez commencer par référencer la ressource une fois dans le modèle pour la créer, puis référencer la ressource à l’aide du même nom pour la mettre à jour ultérieurement.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="2d1b4-109">Toutefois, si deux ressources ont le même nom dans un modèle, Resource Manager lève une exception.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="2d1b4-110">Pour éviter cette erreur, spécifiez la ressource mise à jour dans un second modèle qui est lié ou inclus en tant que sous-modèle à l’aide du type de ressource `Microsoft.Resources/deployments`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="2d1b4-111">Ensuite, vous devez spécifier le nom de la propriété existante à modifier ou un nouveau nom de propriété à ajouter dans le modèle imbriqué.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="2d1b4-112">Vous devez également spécifier les propriétés initiales et leurs valeurs d’origine.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="2d1b4-113">Si vous ne fournissez pas les propriétés et valeurs d’origine, Resource Manager part du principe que vous souhaitez créer une nouvelle ressource et supprime la ressource initiale.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="2d1b4-114">Exemple de modèle</span><span class="sxs-lookup"><span data-stu-id="2d1b4-114">Example template</span></span>

<span data-ttu-id="2d1b4-115">Examinons un exemple de modèle illustrant cette approche.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="2d1b4-116">Notre modèle déploie un réseau virtuel nommé `firstVNet` qui comporte un sous-réseau appelé `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-116">Our template deploys a virtual network  named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="2d1b4-117">Puis il déploie une interface de réseau virtuel nommée `nic1` et l’associe à notre sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="2d1b4-118">Ensuite, une ressource de déploiement nommée `updateVNet` inclut un modèle imbriqué qui met à jour notre ressource `firstVNet` en ajoutant un second sous-réseau appelé `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span> 

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {},
  "resources": [
      {
      "apiVersion": "2016-03-30",
      "name": "firstVNet",
      "location":"[resourceGroup().location]",
      "type": "Microsoft.Network/virtualNetworks",
      "properties": {
          "addressSpace":{"addressPrefixes": [
              "10.0.0.0/22"
          ]},
          "subnets":[              
              {
                  "name":"firstSubnet",
                  "properties":{
                    "addressPrefix":"10.0.0.0/24"
                  }
              }
            ]
      }
    },
    {
        "apiVersion": "2015-06-15",
        "type":"Microsoft.Network/networkInterfaces",
        "name":"nic1",
        "location":"[resourceGroup().location]",
        "dependsOn": [
            "firstVNet"
        ],
        "properties": {
            "ipConfigurations":[
                {
                    "name":"ipconfig1",
                    "properties": {
                        "privateIPAllocationMethod":"Dynamic",
                        "subnet": {
                            "id": "[concat(resourceId('Microsoft.Network/virtualNetworks','firstVNet'),'/subnets/firstSubnet')]"
                        }
                    }
                }
            ]
        }
    },
    {
      "apiVersion": "2015-01-01",
      "type": "Microsoft.Resources/deployments",
      "name": "updateVNet",
      "dependsOn": [
          "nic1"
      ],
      "properties": {
        "mode": "Incremental",
        "parameters": {},
        "template": {
          "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "parameters": {},
          "variables": {},
          "resources": [
              {
                  "apiVersion": "2016-03-30",
                  "name": "firstVNet",
                  "location":"[resourceGroup().location]",
                  "type": "Microsoft.Network/virtualNetworks",
                  "properties": {
                      "addressSpace": "[reference('firstVNet').addressSpace]",
                      "subnets":[
                          {
                              "name":"[reference('firstVNet').subnets[0].name]",
                              "properties":{
                                  "addressPrefix":"[reference('firstVNet').subnets[0].properties.addressPrefix]"
                                  }
                          },
                          {
                              "name":"secondSubnet",
                              "properties":{
                                  "addressPrefix":"10.0.1.0/24"
                                  }
                          }
                     ]
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

<span data-ttu-id="2d1b4-119">Commençons par examiner l’objet de ressource relatif à notre ressource `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="2d1b4-120">Notez que nous spécifions de nouveau les paramètres de notre ressource `firstVNet` dans un modèle imbriqué &mdash; étant donné que Resource Manager n’autorise pas l’utilisation du même nom de déploiement dans le même modèle et que les modèles imbriqués sont considérés comme des modèles distincts.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="2d1b4-121">En spécifiant une nouvelle fois nos valeurs pour notre ressource `firstSubnet`, nous indiquons à Resource Manager de mettre à jour la ressource existante au lieu de la supprimer et de la redéployer.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="2d1b4-122">Pour finir, nos nouveaux paramètres pour `secondSubnet` sont sélectionnés dans le cadre de cette mise à jour.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="2d1b4-123">Essayer le modèle</span><span class="sxs-lookup"><span data-stu-id="2d1b4-123">Try the template</span></span>

<span data-ttu-id="2d1b4-124">Un exemple de modèle est disponible sur [GitHub][github].</span><span class="sxs-lookup"><span data-stu-id="2d1b4-124">An example template is available on [GitHub][github].</span></span> <span data-ttu-id="2d1b4-125">Pour déployer le modèle, exécutez les commandes [Azure CLI][cli] suivantes :</span><span class="sxs-lookup"><span data-stu-id="2d1b4-125">To deploy the template, run the following [Azure CLI][cli] commands:</span></span>

```bash
az group create --location <location> --name <resource-group-name>
az group deployment create -g <resource-group-name> \
    --template-uri https://raw.githubusercontent.com/mspnp/template-examples/master/example1-update/deploy.json
```

<span data-ttu-id="2d1b4-126">Une fois le déploiement terminé, ouvrez le groupe de ressources que vous avez spécifié dans le portail.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-126">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="2d1b4-127">Vous voyez un réseau virtuel nommé `firstVNet` et une interface de réseau virtuel appelée `nic1`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-127">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="2d1b4-128">Cliquez sur `firstVNet`, puis sur `subnets`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-128">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="2d1b4-129">Vous voyez `firstSubnet` qui a été créé à l’origine et vous voyez `secondSubnet` qui a été ajouté dans la ressource `updateVNet`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-129">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span> 

![Sous-réseau d’origine et sous-réseau mis à jour](../_images/firstVNet-subnets.png)

<span data-ttu-id="2d1b4-131">Ensuite, revenez au groupe de ressources, cliquez sur `nic1`, puis cliquez sur `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-131">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="2d1b4-132">Dans la section `IP configurations`, `subnet` est défini sur `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-132">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span> 

![Paramètres de configuration IP nic1](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="2d1b4-134">Le `firstVNet` d’origine a été mis à jour au lieu d’être recréé.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-134">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="2d1b4-135">Si `firstVNet` avait été recréé, `nic1` ne serait pas être associé à `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-135">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2d1b4-136">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="2d1b4-136">Next steps</span></span>

* <span data-ttu-id="2d1b4-137">Découvrez comment déployer une ressource basée sur une condition, par exemple en fonction de la présence d’une valeur de paramètre.</span><span class="sxs-lookup"><span data-stu-id="2d1b4-137">Learn how deploy a resource based on a condition, such as whether a parameter value is present.</span></span> <span data-ttu-id="2d1b4-138">Consultez [Déployer une ressource de manière conditionnelle dans un modèle Azure Resource Manager](./conditional-deploy.md).</span><span class="sxs-lookup"><span data-stu-id="2d1b4-138">See [Conditionally deploy a resource in an Azure Resource Manager template](./conditional-deploy.md).</span></span>

[cli]: /cli/azure/?view=azure-cli-latest
[github]: https://github.com/mspnp/template-examples