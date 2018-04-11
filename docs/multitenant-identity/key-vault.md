---
title: Utiliser Key Vault pour protéger les secrets d’application
description: Comment utiliser le service Key Vault pour stocker les secrets d’application
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 45d1564c255f2450f68c5e92ebe0d7de0c40ae31
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2017
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="e9d87-103">Utiliser Azure Key Vault pour protéger les secrets d’application</span><span class="sxs-lookup"><span data-stu-id="e9d87-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="e9d87-104">[![GitHub](../_images/github.png) Exemple de code][sample application]</span><span class="sxs-lookup"><span data-stu-id="e9d87-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="e9d87-105">Il est courant que les paramètres d’une application revêtent un caractère sensible et doivent être protégés, par exemple :</span><span class="sxs-lookup"><span data-stu-id="e9d87-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="e9d87-106">Chaînes de connexion de base de données</span><span class="sxs-lookup"><span data-stu-id="e9d87-106">Database connection strings</span></span>
* <span data-ttu-id="e9d87-107">Mot de passe</span><span class="sxs-lookup"><span data-stu-id="e9d87-107">Passwords</span></span>
* <span data-ttu-id="e9d87-108">Clés de chiffrement</span><span class="sxs-lookup"><span data-stu-id="e9d87-108">Cryptographic keys</span></span>

<span data-ttu-id="e9d87-109">Par sécurité, il est conseillé de ne jamais stocker ces secrets dans le contrôle de code source.</span><span class="sxs-lookup"><span data-stu-id="e9d87-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="e9d87-110">Elles peuvent fuiter très facilement, même si votre dépôt de code source est privé.</span><span class="sxs-lookup"><span data-stu-id="e9d87-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="e9d87-111">De plus, il ne s’agit pas seulement de les préserver d’un accès par le grand public.</span><span class="sxs-lookup"><span data-stu-id="e9d87-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="e9d87-112">Sur les projets volumineux, vous souhaiterez peut-être restreindre l’accès aux données de production secrètes à certains développeurs et opérateurs.</span><span class="sxs-lookup"><span data-stu-id="e9d87-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="e9d87-113">(Les paramètres sont différents pour les environnements de test et de développement.)</span><span class="sxs-lookup"><span data-stu-id="e9d87-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="e9d87-114">Une option plus sûre consiste à stocker ces secrets dans [Azure Key Vault][KeyVault].</span><span class="sxs-lookup"><span data-stu-id="e9d87-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="e9d87-115">Ce service hébergé dans le cloud assure la gestion des clés de chiffrement et d’autres secrets.</span><span class="sxs-lookup"><span data-stu-id="e9d87-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="e9d87-116">Cet article explique comment l’utiliser pour stocker les paramètres de configuration de votre application.</span><span class="sxs-lookup"><span data-stu-id="e9d87-116">This article shows how to use Key Vault to store configuration settings for you app.</span></span>

<span data-ttu-id="e9d87-117">Dans l’application [Tailspin Surveys][Surveys], les paramètres suivants sont secrets :</span><span class="sxs-lookup"><span data-stu-id="e9d87-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="e9d87-118">La chaîne de connexion de base de données.</span><span class="sxs-lookup"><span data-stu-id="e9d87-118">The database connection string.</span></span>
* <span data-ttu-id="e9d87-119">La chaîne de connexion Redis.</span><span class="sxs-lookup"><span data-stu-id="e9d87-119">The Redis connection string.</span></span>
* <span data-ttu-id="e9d87-120">Le secret client de l’application web.</span><span class="sxs-lookup"><span data-stu-id="e9d87-120">The client secret for the web application.</span></span>

<span data-ttu-id="e9d87-121">L’application Surveys charge les paramètres de configuration à partir des emplacements suivants :</span><span class="sxs-lookup"><span data-stu-id="e9d87-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="e9d87-122">Le fichier appsettings.json</span><span class="sxs-lookup"><span data-stu-id="e9d87-122">The appsettings.json file</span></span>
* <span data-ttu-id="e9d87-123">Le [magasin des secrets utilisateur][user-secrets] (environnement de développement uniquement ; à des fins de test)</span><span class="sxs-lookup"><span data-stu-id="e9d87-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="e9d87-124">L’environnement d’hébergement (paramètres des applications web Azure)</span><span class="sxs-lookup"><span data-stu-id="e9d87-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="e9d87-125">Le service Key Vault (quand il est activé)</span><span class="sxs-lookup"><span data-stu-id="e9d87-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="e9d87-126">Chacun de ces paramètres se substituant aux précédents, les paramètres stockés dans Key Vault sont prioritaires.</span><span class="sxs-lookup"><span data-stu-id="e9d87-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="e9d87-127">Par défaut, le fournisseur de configuration du coffre de clés est désactivé.</span><span class="sxs-lookup"><span data-stu-id="e9d87-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="e9d87-128">Il n’est pas nécessaire pour exécuter l’application localement.</span><span class="sxs-lookup"><span data-stu-id="e9d87-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="e9d87-129">Vous devez l’activer dans un environnement de production.</span><span class="sxs-lookup"><span data-stu-id="e9d87-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="e9d87-130">Au démarrage, l’application lit les paramètres de chaque fournisseur de configuration enregistré et les utilise pour remplir un objet d’options fortement typé.</span><span class="sxs-lookup"><span data-stu-id="e9d87-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="e9d87-131">Pour plus d’informations, consultez [Utilisation des options et des objets de configuration][options].</span><span class="sxs-lookup"><span data-stu-id="e9d87-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="e9d87-132">Configuration du coffre de clés dans l’application Surveys</span><span class="sxs-lookup"><span data-stu-id="e9d87-132">Setting up Key Vault in the Surveys app</span></span>
<span data-ttu-id="e9d87-133">Prérequis :</span><span class="sxs-lookup"><span data-stu-id="e9d87-133">Prerequisites:</span></span>

* <span data-ttu-id="e9d87-134">Installez les [applets de commande Azure Resource Manager][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="e9d87-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="e9d87-135">Configurez l’application Surveys, comme indiqué dans [Exécuter l’application Surveys][readme].</span><span class="sxs-lookup"><span data-stu-id="e9d87-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="e9d87-136">Procédure générale :</span><span class="sxs-lookup"><span data-stu-id="e9d87-136">High-level steps:</span></span>

1. <span data-ttu-id="e9d87-137">Configurez un utilisateur admin dans le client.</span><span class="sxs-lookup"><span data-stu-id="e9d87-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="e9d87-138">Configurez un certificat client.</span><span class="sxs-lookup"><span data-stu-id="e9d87-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="e9d87-139">Création d’un coffre de clés</span><span class="sxs-lookup"><span data-stu-id="e9d87-139">Create a key vault.</span></span>
4. <span data-ttu-id="e9d87-140">Ajoutez des paramètres de configuration à votre coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="e9d87-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="e9d87-141">Supprimez les marques de commentaire du code qui active le coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="e9d87-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="e9d87-142">Mettez à jour les clés secrètes de l’utilisateur de l’application.</span><span class="sxs-lookup"><span data-stu-id="e9d87-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="e9d87-143">Configuration d’un utilisateur admin</span><span class="sxs-lookup"><span data-stu-id="e9d87-143">Set up an admin user</span></span>
> [!NOTE]
> <span data-ttu-id="e9d87-144">Pour créer un coffre de clés, vous devez utiliser un compte qui peut gérer votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="e9d87-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="e9d87-145">De même, toute application que vous autorisez à lire dans le coffre de clés doit être inscrite dans le même locataire que ce compte.</span><span class="sxs-lookup"><span data-stu-id="e9d87-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>
> 
> 

<span data-ttu-id="e9d87-146">Dans cette étape, vous allez vous assurer que vous pouvez créer un coffre de clés lorsque vous êtes connecté en tant qu’utilisateur du client où l’application Surveys est inscrite.</span><span class="sxs-lookup"><span data-stu-id="e9d87-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="e9d87-147">Créez un utilisateur administrateur au sein du locataire Azure AD où l’application Surveys est inscrite.</span><span class="sxs-lookup"><span data-stu-id="e9d87-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="e9d87-148">Connectez-vous au [portail Azure][azure-portal].</span><span class="sxs-lookup"><span data-stu-id="e9d87-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="e9d87-149">Sélectionnez le client Azure AD où votre application est inscrite.</span><span class="sxs-lookup"><span data-stu-id="e9d87-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="e9d87-150">Cliquez sur **Autres services** > **SÉCURITÉ + IDENTITÉ** > **Azure Active Directory** > **Utilisateurs et groupes**  > **Tous les utilisateurs**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="e9d87-151">Dans la partie supérieure du portail, cliquez sur **Nouvel utilisateur**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="e9d87-152">Complétez les champs et assignez l’utilisateur au rôle d’annuaire **Administrateur général**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="e9d87-153">Cliquez sur **Créer**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-153">Click **Create**.</span></span>

![Utilisateur administrateur général](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="e9d87-155">Assignez maintenant cet utilisateur comme propriétaire d’abonnement.</span><span class="sxs-lookup"><span data-stu-id="e9d87-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="e9d87-156">Dans le menu Hub, sélectionnez **Abonnements**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="e9d87-157">Sélectionnez l’abonnement auquel l’administrateur doit accéder.</span><span class="sxs-lookup"><span data-stu-id="e9d87-157">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="e9d87-158">Dans le panneau de l’abonnement, sélectionnez **Contrôle d’accès (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-158">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="e9d87-159">Cliquez sur **Ajouter**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-159">Click **Add**.</span></span>
4. <span data-ttu-id="e9d87-160">Sous **Rôle**, sélectionnez **Propriétaire**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-160">Under **Role**, select **Owner**.</span></span>
5. <span data-ttu-id="e9d87-161">Tapez l’adresse e-mail de l’utilisateur à ajouter comme propriétaire.</span><span class="sxs-lookup"><span data-stu-id="e9d87-161">Type the email address of the user you want to add as owner.</span></span>
6. <span data-ttu-id="e9d87-162">Sélectionnez l’utilisateur et cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-162">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="e9d87-163">Configuration d’un certificat client</span><span class="sxs-lookup"><span data-stu-id="e9d87-163">Set up a client certificate</span></span>
1. <span data-ttu-id="e9d87-164">Exécutez le script PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] comme suit :</span><span class="sxs-lookup"><span data-stu-id="e9d87-164">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="e9d87-165">Pour le paramètre `Subject` , entrez un nom, comme « surveysapp ».</span><span class="sxs-lookup"><span data-stu-id="e9d87-165">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="e9d87-166">Le script génère un certificat auto-signé et le stocke dans le magasin de certificats « Utilisateur actuel/Personnel ».</span><span class="sxs-lookup"><span data-stu-id="e9d87-166">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="e9d87-167">La sortie du script est un fragment JSON.</span><span class="sxs-lookup"><span data-stu-id="e9d87-167">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="e9d87-168">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="e9d87-168">Copy this value.</span></span>

2. <span data-ttu-id="e9d87-169">Dans le [portail Azure][azure-portal], basculez vers le répertoire où est inscrite l’application Surveys en sélectionnant votre compte en haut à droite du portail.</span><span class="sxs-lookup"><span data-stu-id="e9d87-169">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="e9d87-170">Sélectionnez **Azure Active Directory** > **Inscriptions des applications** > Surveys</span><span class="sxs-lookup"><span data-stu-id="e9d87-170">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4.  <span data-ttu-id="e9d87-171">Cliquez sur **Manifeste**, puis sur **Modifier**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-171">Click **Manifest** and then **Edit**.</span></span>

5.  <span data-ttu-id="e9d87-172">Collez la sortie du script dans la propriété `keyCredentials`.</span><span class="sxs-lookup"><span data-stu-id="e9d87-172">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="e9d87-173">Le résultat doit être semblable à ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="e9d87-173">It should look similar to the following:</span></span>
        
    ```json
    "keyCredentials": [
        {
        "type": "AsymmetricX509Cert",
        "usage": "Verify",
        "keyId": "29d4f7db-0539-455e-b708-....",
        "customKeyIdentifier": "ZEPpP/+KJe2fVDBNaPNOTDoJMac=",
        "value": "MIIDAjCCAeqgAwIBAgIQFxeRiU59eL.....
        }
    ],
    ```          

6. <span data-ttu-id="e9d87-174">Cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-174">Click **Save**.</span></span>  

7. <span data-ttu-id="e9d87-175">Répétez les étapes 3 à 6 pour ajouter le même fragment JSON au manifeste d’application de l’API web (Surveys.WebAPI).</span><span class="sxs-lookup"><span data-stu-id="e9d87-175">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="e9d87-176">Dans la fenêtre PowerShell, exécutez la commande suivante pour obtenir l’empreinte numérique du certificat.</span><span class="sxs-lookup"><span data-stu-id="e9d87-176">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>
   
    ```
    certutil -store -user my [subject]
    ```
    
    <span data-ttu-id="e9d87-177">Pour `[subject]`, utilisez la valeur que vous avez spécifiée pour Subject dans le script PowerShell.</span><span class="sxs-lookup"><span data-stu-id="e9d87-177">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="e9d87-178">L’empreinte numérique est répertoriée sous « Cert Hash(sha1) ».</span><span class="sxs-lookup"><span data-stu-id="e9d87-178">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="e9d87-179">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="e9d87-179">Copy this value.</span></span> <span data-ttu-id="e9d87-180">Vous utiliserez l’empreinte numérique ultérieurement.</span><span class="sxs-lookup"><span data-stu-id="e9d87-180">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="e9d87-181">Création d’un coffre de clés</span><span class="sxs-lookup"><span data-stu-id="e9d87-181">Create a key vault</span></span>
1. <span data-ttu-id="e9d87-182">Exécutez le script PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] comme suit :</span><span class="sxs-lookup"><span data-stu-id="e9d87-182">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```
   
    <span data-ttu-id="e9d87-183">Lorsque vous êtes invité à entrer vos informations d’identification, connectez-vous avec les informations de l’utilisateur Azure AD que vous avez créé précédemment.</span><span class="sxs-lookup"><span data-stu-id="e9d87-183">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="e9d87-184">Le script crée un groupe de ressources et un coffre de clés au sein de ce groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="e9d87-184">The script creates a new resource group, and a new key vault within that resource group.</span></span> 
   
2. <span data-ttu-id="e9d87-185">Exécutez à nouveau SetupKeyVault.ps comme suit :</span><span class="sxs-lookup"><span data-stu-id="e9d87-185">Run SetupKeyVault.ps again as follows:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```
   
    <span data-ttu-id="e9d87-186">Définissez les valeurs des paramètres suivantes :</span><span class="sxs-lookup"><span data-stu-id="e9d87-186">Set the following parameter values:</span></span>
   
       * <span data-ttu-id="e9d87-187">key vault name = Le nom que vous avez affecté au coffre de clés à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="e9d87-187">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="e9d87-188">Surveys app ID = ID de l’application web Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d87-188">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="e9d87-189">Surveys.WebApi app ID = ID de l’application Surveys.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="e9d87-189">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>
         
    <span data-ttu-id="e9d87-190">Exemple :</span><span class="sxs-lookup"><span data-stu-id="e9d87-190">Example:</span></span>
     
    ```
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```
    
    <span data-ttu-id="e9d87-191">Ce script autorise l’application web et l’API web à extraire les données secrètes de votre coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="e9d87-191">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="e9d87-192">Pour plus d’informations, consultez [Prise en main du coffre de clés Azure](/azure/key-vault/key-vault-get-started/).</span><span class="sxs-lookup"><span data-stu-id="e9d87-192">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="e9d87-193">Ajout de paramètres de configuration à votre coffre de clés</span><span class="sxs-lookup"><span data-stu-id="e9d87-193">Add configuration settings to your key vault</span></span>
1. <span data-ttu-id="e9d87-194">Exécutez SetupKeyVault.ps comme suit :</span><span class="sxs-lookup"><span data-stu-id="e9d87-194">Run SetupKeyVault.ps as follows::</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true" 
    ```
    <span data-ttu-id="e9d87-195">où </span><span class="sxs-lookup"><span data-stu-id="e9d87-195">where</span></span>
   
   * <span data-ttu-id="e9d87-196">key vault name = Le nom que vous avez affecté au coffre de clés à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="e9d87-196">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="e9d87-197">Redis DNS name = Le nom DNS de votre instance de cache Redis.</span><span class="sxs-lookup"><span data-stu-id="e9d87-197">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="e9d87-198">Redis access key = la clé d’accès pour votre instance de cache Redis.</span><span class="sxs-lookup"><span data-stu-id="e9d87-198">Redis access key = The access key for your Redis cache instance.</span></span>
     
2. <span data-ttu-id="e9d87-199">À ce stade, il est judicieux de vérifier si vous avez stocké correctement les clés secrètes dans le coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="e9d87-199">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="e9d87-200">Exécutez la commande PowerShell suivante :</span><span class="sxs-lookup"><span data-stu-id="e9d87-200">Run the following PowerShell command:</span></span>
   
    ```
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="e9d87-201">Réexécutez SetupKeyVault.ps pour ajouter la chaîne de connexion de base de données :</span><span class="sxs-lookup"><span data-stu-id="e9d87-201">Run SetupKeyVault.ps again to add the database connection string:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```
   
    <span data-ttu-id="e9d87-202">où `<<DB connection string>>` est la valeur de la chaîne de connexion de base de données.</span><span class="sxs-lookup"><span data-stu-id="e9d87-202">where `<<DB connection string>>` is the value of the database connection string.</span></span>
   
    <span data-ttu-id="e9d87-203">Pour effectuer un test avec la base de données locale, copiez la chaîne de connexion à partir du fichier Tailspin.Surveys.Web/appsettings.json.</span><span class="sxs-lookup"><span data-stu-id="e9d87-203">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="e9d87-204">Si vous faites cela, veillez à remplacer la double barre oblique inverse (« \\\\ ») par une simple barre oblique inverse.</span><span class="sxs-lookup"><span data-stu-id="e9d87-204">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="e9d87-205">La double barre oblique inverse est un caractère d’échappement dans le fichier JSON.</span><span class="sxs-lookup"><span data-stu-id="e9d87-205">The double backslash is an escape character in the JSON file.</span></span>
   
    <span data-ttu-id="e9d87-206">Exemple :</span><span class="sxs-lookup"><span data-stu-id="e9d87-206">Example:</span></span>
   
    ```
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true" 
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="e9d87-207">Suppression des marques de commentaire du code qui active le coffre de clés</span><span class="sxs-lookup"><span data-stu-id="e9d87-207">Uncomment the code that enables Key Vault</span></span>
1. <span data-ttu-id="e9d87-208">Ouvrez la solution Tailspin.Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d87-208">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="e9d87-209">Dans Tailspin.Surveys.Web/Startup.cs, localisez le bloc de code suivant et supprimez les commentaires.</span><span class="sxs-lookup"><span data-stu-id="e9d87-209">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>
   
    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="e9d87-210">Dans Tailspin.Surveys.Web/Startup.cs, localisez le code qui inscrit `ICredentialService`.</span><span class="sxs-lookup"><span data-stu-id="e9d87-210">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="e9d87-211">Supprimez les commentaires de la ligne qui utilise `CertificateCredentialService`, puis commentez la ligne qui utilise `ClientCredentialService` :</span><span class="sxs-lookup"><span data-stu-id="e9d87-211">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>
   
    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```
   
    <span data-ttu-id="e9d87-212">Cette modification permet à l’application web d’utiliser l’[Assertion de client][client-assertion] pour obtenir des jetons d’accès OAuth.</span><span class="sxs-lookup"><span data-stu-id="e9d87-212">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="e9d87-213">Avec l’assertion de client, vous n’avez pas besoin de secret client OAuth.</span><span class="sxs-lookup"><span data-stu-id="e9d87-213">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="e9d87-214">Vous pouvez aussi stocker le secret client dans le coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="e9d87-214">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="e9d87-215">Cependant, le coffre de clés et l’assertion de client utilisent tous deux un certificat client. Donc, si vous activez le coffre de clés, il est recommandé d’activer aussi l’assertion de client.</span><span class="sxs-lookup"><span data-stu-id="e9d87-215">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="e9d87-216">Mise à jour des données secrètes de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="e9d87-216">Update the user secrets</span></span>
<span data-ttu-id="e9d87-217">Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet Tailspin.Surveys.Web, puis sélectionnez **Gérer les données secrètes de l’utilisateur**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-217">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="e9d87-218">Dans le fichier secrets.json, supprimez le script JSON existant et collez les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="e9d87-218">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

    ```
    {
      "AzureAd": {
        "ClientId": "[Surveys web app client ID]",
        "ClientSecret": "[Surveys web app client secret]",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "[App ID URI of your Surveys.WebAPI application]",
        "Asymmetric": {
          "CertificateThumbprint": "[certificate thumbprint. Example: 105b2ff3bc842c53582661716db1b7cdc6b43ec9]",
          "StoreName": "My",
          "StoreLocation": "CurrentUser",
          "ValidationRequired": "false"
        }
      },
      "KeyVault": {
        "Name": "[key vault name]"
      }
    }
    ```

<span data-ttu-id="e9d87-219">Remplacez les entrées entre [crochets] par les valeurs correctes.</span><span class="sxs-lookup"><span data-stu-id="e9d87-219">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="e9d87-220">`AzureAd:ClientId`: L’ID client de l’application Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d87-220">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="e9d87-221">`AzureAd:ClientSecret`: La clé que vous avez générée au moment d’inscrire l’application Surveys dans Azure AD.</span><span class="sxs-lookup"><span data-stu-id="e9d87-221">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="e9d87-222">`AzureAd:WebApiResourceId`: L’URI ID d’application que vous avez spécifié lorsque vous avez créé l’application Surveys.WebAPI dans Azure AD.</span><span class="sxs-lookup"><span data-stu-id="e9d87-222">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="e9d87-223">`Asymmetric:CertificateThumbprint`: L’empreinte numérique de certificat que vous avez obtenue précédemment, lorsque vous avez créé le certificat client.</span><span class="sxs-lookup"><span data-stu-id="e9d87-223">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="e9d87-224">`KeyVault:Name`: Le nom de votre coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="e9d87-224">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="e9d87-225">`Asymmetric:ValidationRequired` a la valeur false, car le certificat que vous avez créé précédemment n’a pas été signé par une autorité de certification racine.</span><span class="sxs-lookup"><span data-stu-id="e9d87-225">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="e9d87-226">En production, utilisez un certificat signé par une autorité de certification racine, puis affectez à `ValidationRequired` la valeur true.</span><span class="sxs-lookup"><span data-stu-id="e9d87-226">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>
> 
> 

<span data-ttu-id="e9d87-227">Enregistrez le fichier secrets.json mis à jour.</span><span class="sxs-lookup"><span data-stu-id="e9d87-227">Save the updated secrets.json file.</span></span>

<span data-ttu-id="e9d87-228">Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet Tailspin.Surveys.WebApi, puis sélectionnez **Gérer les données secrètes de l’utilisateur**.</span><span class="sxs-lookup"><span data-stu-id="e9d87-228">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="e9d87-229">Supprimez le script JSON existant et collez les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="e9d87-229">Delete the existing JSON and paste in the following:</span></span>

```
{
  "AzureAd": {
    "ClientId": "[Surveys.WebAPI client ID]",
    "WebApiResourceId": "https://tailspin5.onmicrosoft.com/surveys.webapi",
    "Asymmetric": {
      "CertificateThumbprint": "[certificate thumbprint]",
      "StoreName": "My",
      "StoreLocation": "CurrentUser",
      "ValidationRequired": "false"
    }
  },
  "KeyVault": {
    "Name": "[key vault name]"
  }
}
```

<span data-ttu-id="e9d87-230">Remplacez les entrées entre [crochets] et enregistrez le fichier secrets.json.</span><span class="sxs-lookup"><span data-stu-id="e9d87-230">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="e9d87-231">Pour l’API web, veillez à utiliser l’ID client de l’application Surveys.WebAPI et non de l’application Surveys.</span><span class="sxs-lookup"><span data-stu-id="e9d87-231">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>
> 
> 

<span data-ttu-id="e9d87-232">[**Suivant**][adfs]</span><span class="sxs-lookup"><span data-stu-id="e9d87-232">[**Next**][adfs]</span></span>

<!-- Links -->
[adfs]: ./adfs.md
[authorize-app]: /azure/key-vault/key-vault-get-started//#authorize
[azure-portal]: https://portal.azure.com
[azure-rm-cmdlets]: https://msdn.microsoft.com/library/mt125356.aspx
[client-assertion]: client-assertion.md
[configuration]: /aspnet/core/fundamentals/configuration
[KeyVault]: https://azure.microsoft.com/services/key-vault/
[key-tags]: https://msdn.microsoft.com/library/azure/dn903623.aspx#BKMK_Keytags
[Microsoft.Azure.KeyVault]: https://www.nuget.org/packages/Microsoft.Azure.KeyVault/
[options]: /aspnet/core/fundamentals/configuration#using-options-and-configuration-objects
[readme]: ./run-the-app.md
[Setup-KeyVault]: https://github.com/mspnp/multitenant-saas-guidance/blob/master/scripts/Setup-KeyVault.ps1
[Surveys]: tailspin.md
[user-secrets]: http://go.microsoft.com/fwlink/?LinkID=532709
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
