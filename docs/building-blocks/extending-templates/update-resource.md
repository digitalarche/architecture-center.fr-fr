---
title: Mettre à jour une ressource dans un modèle Azure Resource Manager
description: Explique comment étendre les fonctionnalités des modèles Azure Resource Manager afin de mettre à jour une ressource
author: petertay
ms.date: 06/09/2017
ms.openlocfilehash: f235f0b4d54d65ccc2fa67876916e922d75f6d07
ms.sourcegitcommit: 94d50043db63416c4d00cebe927a0c88f78c3219
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/28/2018
ms.locfileid: "47429030"
---
# <a name="update-a-resource-in-an-azure-resource-manager-template"></a><span data-ttu-id="e9b9a-103">Mettre à jour une ressource dans un modèle Azure Resource Manager</span><span class="sxs-lookup"><span data-stu-id="e9b9a-103">Update a resource in an Azure Resource Manager template</span></span>

<span data-ttu-id="e9b9a-104">Il existe certains scénarios dans lesquels vous devez mettre à jour une ressource pendant un déploiement.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-104">There are some scenarios in which you need to update a resource during a deployment.</span></span> <span data-ttu-id="e9b9a-105">Vous pouvez rencontrer ce scénario lorsque vous ne pouvez pas spécifier toutes les propriétés d’une ressource jusqu’à ce que les autres ressources dépendantes aient été créées.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-105">You might encounter this scenario when you cannot specify all the properties for a resource until other, dependent resources are created.</span></span> <span data-ttu-id="e9b9a-106">Par exemple, si vous créez un pool principal pour un équilibrage de charge, vous pouvez mettre à jour les interfaces réseau (NIC) sur vos machines virtuelles pour les inclure dans le pool principal.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-106">For example, if you create a backend pool for a load balancer, you might update the network interfaces (NICs) on your virtual machines (VMs) to include them in the backend pool.</span></span> <span data-ttu-id="e9b9a-107">Alors que Resource Manager prend en charge la mise à jour des ressources lors du déploiement, vous devez concevoir votre modèle correctement pour éviter les erreurs et garantir que le déploiement est traité comme une mise à jour.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-107">And while Resource Manager supports updating resources during deployment, you must design your template correctly to avoid errors and to ensure the deployment is handled as an update.</span></span>

<span data-ttu-id="e9b9a-108">Vous devez commencer par référencer la ressource une fois dans le modèle pour la créer, puis référencer la ressource à l’aide du même nom pour la mettre à jour ultérieurement.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-108">First, you must reference the resource once in the template to create it and then reference the resource by the same name to update it later.</span></span> <span data-ttu-id="e9b9a-109">Toutefois, si deux ressources ont le même nom dans un modèle, Resource Manager lève une exception.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-109">However, if two resources have the same name in a template, Resource Manager throws an exception.</span></span> <span data-ttu-id="e9b9a-110">Pour éviter cette erreur, spécifiez la ressource mise à jour dans un second modèle qui est lié ou inclus en tant que sous-modèle à l’aide du type de ressource `Microsoft.Resources/deployments`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-110">To avoid this error, specify the updated resource in a second template that's either linked or included as a subtemplate using the `Microsoft.Resources/deployments` resource type.</span></span>

<span data-ttu-id="e9b9a-111">Ensuite, vous devez spécifier le nom de la propriété existante à modifier ou un nouveau nom de propriété à ajouter dans le modèle imbriqué.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-111">Second, you must either specify the name of the existing property to change or a new name for a property to add in the nested template.</span></span> <span data-ttu-id="e9b9a-112">Vous devez également spécifier les propriétés initiales et leurs valeurs d’origine.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-112">You must also specify the original properties and their original values.</span></span> <span data-ttu-id="e9b9a-113">Si vous ne fournissez pas les propriétés et valeurs d’origine, Resource Manager part du principe que vous souhaitez créer une nouvelle ressource et supprime la ressource initiale.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-113">If you fail to provide the original properties and values, Resource Manager assumes you want to create a new resource and deletes the original resource.</span></span>

## <a name="example-template"></a><span data-ttu-id="e9b9a-114">Exemple de modèle</span><span class="sxs-lookup"><span data-stu-id="e9b9a-114">Example template</span></span>

<span data-ttu-id="e9b9a-115">Examinons un exemple de modèle illustrant cette approche.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-115">Let's look at an example template that demonstrates this.</span></span> <span data-ttu-id="e9b9a-116">Notre modèle déploie un réseau virtuel nommé `firstVNet` qui comporte un sous-réseau appelé `firstSubnet`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-116">Our template deploys a virtual network  named `firstVNet` that has one subnet named `firstSubnet`.</span></span> <span data-ttu-id="e9b9a-117">Puis il déploie une interface de réseau virtuel nommée `nic1` et l’associe à notre sous-réseau.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-117">It then deploys a virtual network interface (NIC) named `nic1` and associates it with our subnet.</span></span> <span data-ttu-id="e9b9a-118">Ensuite, une ressource de déploiement nommée `updateVNet` inclut un modèle imbriqué qui met à jour notre ressource `firstVNet` en ajoutant un second sous-réseau appelé `secondSubnet`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-118">Then, a deployment resource named `updateVNet` includes a nested template that updates our `firstVNet` resource by adding a second subnet named `secondSubnet`.</span></span> 

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

<span data-ttu-id="e9b9a-119">Commençons par examiner l’objet de ressource relatif à notre ressource `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-119">Let's take a look at the resource object for our `firstVNet` resource first.</span></span> <span data-ttu-id="e9b9a-120">Notez que nous spécifions de nouveau les paramètres de notre ressource `firstVNet` dans un modèle imbriqué &mdash; étant donné que Resource Manager n’autorise pas l’utilisation du même nom de déploiement dans le même modèle et que les modèles imbriqués sont considérés comme des modèles distincts.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-120">Notice that we respecify the settings for our `firstVNet` in a nested template&mdash;this is because Resource Manager doesn't allow the same deployment name within the same template and nested templates are considered to be a different template.</span></span> <span data-ttu-id="e9b9a-121">En spécifiant une nouvelle fois nos valeurs pour notre ressource `firstSubnet`, nous indiquons à Resource Manager de mettre à jour la ressource existante au lieu de la supprimer et de la redéployer.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-121">By respecifying our values for our `firstSubnet` resource, we are telling Resource Manager to update the existing resource instead of deleting it and redeploying it.</span></span> <span data-ttu-id="e9b9a-122">Pour finir, nos nouveaux paramètres pour `secondSubnet` sont sélectionnés dans le cadre de cette mise à jour.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-122">Finally, our new settings for `secondSubnet` are picked up during this update.</span></span>

## <a name="try-the-template"></a><span data-ttu-id="e9b9a-123">Essayer le modèle</span><span class="sxs-lookup"><span data-stu-id="e9b9a-123">Try the template</span></span>

<span data-ttu-id="e9b9a-124">Si vous souhaitez effectuer des tests avec ce modèle, procédez comme suit :</span><span class="sxs-lookup"><span data-stu-id="e9b9a-124">If you would like to experiment with this template, follow these steps:</span></span>

1.  <span data-ttu-id="e9b9a-125">Accédez au portail Azure, sélectionnez l’icône **+**, puis recherchez le type de ressource **déploiement de modèle** et sélectionnez-le.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-125">Go to the Azure portal, select the **+** icon, and search for the **template deployment** resource type, and select it.</span></span>
2.  <span data-ttu-id="e9b9a-126">Accédez à la page **déploiement de modèle**, puis sélectionnez le bouton **créer**.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-126">Navigate to the **template deployment** page, select the **create** button.</span></span> <span data-ttu-id="e9b9a-127">Ce bouton ouvre le panneau **déploiement personnalisé**.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-127">This button opens the **custom deployment** blade.</span></span>
3.  <span data-ttu-id="e9b9a-128">Sélectionnez l’icône **éditer**.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-128">Select the **edit** icon.</span></span>
4.  <span data-ttu-id="e9b9a-129">Supprimez le modèle vide.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-129">Delete the empty template.</span></span>
5.  <span data-ttu-id="e9b9a-130">Copiez et collez l’exemple de modèle dans le volet de droite.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-130">Copy and paste the sample template into the right pane.</span></span>
6.  <span data-ttu-id="e9b9a-131">Sélectionnez le bouton **enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-131">Select the **save** button.</span></span>
7.  <span data-ttu-id="e9b9a-132">Vous revenez au volet **déploiement personnalisé**, mais cette fois-ci, certaines zones de liste déroulante apparaissent.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-132">You return to the **custom deployment** pane, but this time there are some drop-down list boxes.</span></span> <span data-ttu-id="e9b9a-133">Sélectionnez votre abonnement, créez un nouveau groupe de ressources ou utilisez un groupe de ressources existant, puis sélectionnez un emplacement.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-133">Select your subscription, either create new or use existing resource group, and select a location.</span></span> <span data-ttu-id="e9b9a-134">Passez en revue les Conditions générales, puis sélectionnez le bouton **J’accepte**.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-134">Review the terms and conditions, then select the **I agree** button.</span></span>
8.  <span data-ttu-id="e9b9a-135">Sélectionnez le bouton **acheter**.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-135">Select the **purchase** button.</span></span>

<span data-ttu-id="e9b9a-136">Une fois le déploiement terminé, ouvrez le groupe de ressources que vous avez spécifié dans le portail.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-136">Once deployment has finished, open the resource group you specified in the portal.</span></span> <span data-ttu-id="e9b9a-137">Vous voyez un réseau virtuel nommé `firstVNet` et une interface de réseau virtuel appelée `nic1`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-137">You see a virtual network named `firstVNet` and a NIC named `nic1`.</span></span> <span data-ttu-id="e9b9a-138">Cliquez sur `firstVNet`, puis sur `subnets`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-138">Click `firstVNet`, then click `subnets`.</span></span> <span data-ttu-id="e9b9a-139">Vous voyez `firstSubnet` qui a été créé à l’origine et vous voyez `secondSubnet` qui a été ajouté dans la ressource `updateVNet`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-139">You see the `firstSubnet` that was originally created, and you see the `secondSubnet` that was added in the `updateVNet` resource.</span></span> 

![Sous-réseau d’origine et sous-réseau mis à jour](../_images/firstVNet-subnets.png)

<span data-ttu-id="e9b9a-141">Ensuite, revenez au groupe de ressources, cliquez sur `nic1`, puis cliquez sur `IP configurations`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-141">Then, go back to the resource group and click `nic1` then click `IP configurations`.</span></span> <span data-ttu-id="e9b9a-142">Dans la section `IP configurations`, `subnet` est défini sur `firstSubnet (10.0.0.0/24)`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-142">In the `IP configurations` section, the `subnet` is set to `firstSubnet (10.0.0.0/24)`.</span></span> 

![Paramètres de configuration IP nic1](../_images/nic1-ipconfigurations.png)

<span data-ttu-id="e9b9a-144">Le `firstVNet` d’origine a été mis à jour au lieu d’être recréé.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-144">The original `firstVNet` has been updated instead of recreated.</span></span> <span data-ttu-id="e9b9a-145">Si `firstVNet` avait été recréé, `nic1` ne serait pas être associé à `firstVNet`.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-145">If `firstVNet` had been recreated, `nic1` would not be associated with `firstVNet`.</span></span>

## <a name="next-steps"></a><span data-ttu-id="e9b9a-146">Étapes suivantes</span><span class="sxs-lookup"><span data-stu-id="e9b9a-146">Next steps</span></span>

* <span data-ttu-id="e9b9a-147">Cette technique est implémentée dans le [projet de blocs de construction de modèle](https://github.com/mspnp/template-building-blocks) et dans les [architectures de référence Azure](/azure/architecture/reference-architectures/).</span><span class="sxs-lookup"><span data-stu-id="e9b9a-147">This technique is implemented in the [template building blocks project](https://github.com/mspnp/template-building-blocks) and the [Azure reference architectures](/azure/architecture/reference-architectures/).</span></span> <span data-ttu-id="e9b9a-148">Vous pouvez utiliser ces derniers pour créer votre propre architecture ou déployer l’une de nos architectures de référence.</span><span class="sxs-lookup"><span data-stu-id="e9b9a-148">You can use these to create your own architecture or deploy one of our reference architectures.</span></span>
