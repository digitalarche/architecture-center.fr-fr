---
title: Utiliser Key Vault pour protéger les secrets d’application
description: Comment utiliser le service Key Vault pour stocker les secrets d’application.
author: MikeWasson
ms.date: 07/21/2017
ms.topic: guide
ms.service: architecture-center
ms.subservice: reference-architecture
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: client-assertion
ms.openlocfilehash: 6aa8d33da0b2fd41fdc037bac28bca9f7ff09907
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 01/23/2019
ms.locfileid: "54481556"
---
# <a name="use-azure-key-vault-to-protect-application-secrets"></a><span data-ttu-id="19e14-103">Utiliser Azure Key Vault pour protéger les secrets d’application</span><span class="sxs-lookup"><span data-stu-id="19e14-103">Use Azure Key Vault to protect application secrets</span></span>

<span data-ttu-id="19e14-104">[![GitHub](../_images/github.png) Exemple de code][sample application]</span><span class="sxs-lookup"><span data-stu-id="19e14-104">[![GitHub](../_images/github.png) Sample code][sample application]</span></span>

<span data-ttu-id="19e14-105">Il est courant que les paramètres d’une application revêtent un caractère sensible et doivent être protégés, par exemple :</span><span class="sxs-lookup"><span data-stu-id="19e14-105">It's common to have application settings that are sensitive and must be protected, such as:</span></span>

* <span data-ttu-id="19e14-106">Chaînes de connexion de base de données</span><span class="sxs-lookup"><span data-stu-id="19e14-106">Database connection strings</span></span>
* <span data-ttu-id="19e14-107">Mot de passe</span><span class="sxs-lookup"><span data-stu-id="19e14-107">Passwords</span></span>
* <span data-ttu-id="19e14-108">Clés de chiffrement</span><span class="sxs-lookup"><span data-stu-id="19e14-108">Cryptographic keys</span></span>

<span data-ttu-id="19e14-109">Par sécurité, il est conseillé de ne jamais stocker ces données secrètes dans le contrôle de code source.</span><span class="sxs-lookup"><span data-stu-id="19e14-109">As a security best practice, you should never store these secrets in source control.</span></span> <span data-ttu-id="19e14-110">Elles peuvent fuiter très facilement, même si votre dépôt de code source est privé.</span><span class="sxs-lookup"><span data-stu-id="19e14-110">It's too easy for them to leak &mdash; even if your source code repository is private.</span></span> <span data-ttu-id="19e14-111">De plus, il ne s’agit pas seulement de les préserver d’un accès par le grand public.</span><span class="sxs-lookup"><span data-stu-id="19e14-111">And it's not just about keeping secrets from the general public.</span></span> <span data-ttu-id="19e14-112">Sur les projets volumineux, vous souhaiterez peut-être restreindre l’accès aux données de production secrètes à certains développeurs et opérateurs.</span><span class="sxs-lookup"><span data-stu-id="19e14-112">On larger projects, you might want to restrict which developers and operators can access the production secrets.</span></span> <span data-ttu-id="19e14-113">(Les paramètres sont différents pour les environnements de test et de développement.)</span><span class="sxs-lookup"><span data-stu-id="19e14-113">(Settings for test or development environments are different.)</span></span>

<span data-ttu-id="19e14-114">Une option plus sûre consiste à stocker ces secrets dans [Azure Key Vault][KeyVault].</span><span class="sxs-lookup"><span data-stu-id="19e14-114">A more secure option is to store these secrets in [Azure Key Vault][KeyVault].</span></span> <span data-ttu-id="19e14-115">Ce service hébergé dans le cloud assure la gestion des clés de chiffrement et d’autres données secrètes.</span><span class="sxs-lookup"><span data-stu-id="19e14-115">Key Vault is a cloud-hosted service for managing cryptographic keys and other secrets.</span></span> <span data-ttu-id="19e14-116">Cet article explique comment l’utiliser pour stocker les paramètres de configuration de votre application.</span><span class="sxs-lookup"><span data-stu-id="19e14-116">This article shows how to use Key Vault to store configuration settings for your app.</span></span>

<span data-ttu-id="19e14-117">Dans l’application [Tailspin Surveys][Surveys], les paramètres suivants sont secrets :</span><span class="sxs-lookup"><span data-stu-id="19e14-117">In the [Tailspin Surveys][Surveys] application, the following settings are secret:</span></span>

* <span data-ttu-id="19e14-118">la chaîne de connexion de base de données ;</span><span class="sxs-lookup"><span data-stu-id="19e14-118">The database connection string.</span></span>
* <span data-ttu-id="19e14-119">la chaîne de connexion Redis ;</span><span class="sxs-lookup"><span data-stu-id="19e14-119">The Redis connection string.</span></span>
* <span data-ttu-id="19e14-120">la clé secrète client de l’application web.</span><span class="sxs-lookup"><span data-stu-id="19e14-120">The client secret for the web application.</span></span>

<span data-ttu-id="19e14-121">L’application Surveys charge les paramètres de configuration à partir des emplacements suivants :</span><span class="sxs-lookup"><span data-stu-id="19e14-121">The Surveys application loads configuration settings from the following places:</span></span>

* <span data-ttu-id="19e14-122">le fichier appsettings.json ;</span><span class="sxs-lookup"><span data-stu-id="19e14-122">The appsettings.json file</span></span>
* <span data-ttu-id="19e14-123">Le [magasin des secrets utilisateur][user-secrets] (environnement de développement uniquement ; à des fins de test)</span><span class="sxs-lookup"><span data-stu-id="19e14-123">The [user secrets store][user-secrets] (development environment only; for testing)</span></span>
* <span data-ttu-id="19e14-124">l’environnement d’hébergement (paramètres des applications web Azure) ;</span><span class="sxs-lookup"><span data-stu-id="19e14-124">The hosting environment (app settings in Azure web apps)</span></span>
* <span data-ttu-id="19e14-125">Le service Key Vault (quand il est activé)</span><span class="sxs-lookup"><span data-stu-id="19e14-125">Key Vault (when enabled)</span></span>

<span data-ttu-id="19e14-126">Chacun de ces paramètres se substituant aux précédents, les paramètres stockés dans Key Vault sont prioritaires.</span><span class="sxs-lookup"><span data-stu-id="19e14-126">Each of these overrides the previous one, so any settings stored in Key Vault take precedence.</span></span>

> [!NOTE]
> <span data-ttu-id="19e14-127">Par défaut, le fournisseur de configuration du coffre de clés est désactivé.</span><span class="sxs-lookup"><span data-stu-id="19e14-127">By default, the Key Vault configuration provider is disabled.</span></span> <span data-ttu-id="19e14-128">Il n’est pas nécessaire pour exécuter l’application localement.</span><span class="sxs-lookup"><span data-stu-id="19e14-128">It's not needed for running the application locally.</span></span> <span data-ttu-id="19e14-129">Vous devez l’activer dans un environnement de production.</span><span class="sxs-lookup"><span data-stu-id="19e14-129">You would enable it in a production deployment.</span></span>

<span data-ttu-id="19e14-130">Au démarrage, l’application lit les paramètres de chaque fournisseur de configuration enregistré et les utilise pour remplir un objet d’options fortement typé.</span><span class="sxs-lookup"><span data-stu-id="19e14-130">At startup, the application reads settings from every registered configuration provider, and uses them to populate a strongly typed options object.</span></span> <span data-ttu-id="19e14-131">Pour plus d’informations, consultez [Utilisation des options et des objets de configuration][options].</span><span class="sxs-lookup"><span data-stu-id="19e14-131">For more information, see [Using Options and configuration objects][options].</span></span>

## <a name="setting-up-key-vault-in-the-surveys-app"></a><span data-ttu-id="19e14-132">Configuration du coffre de clés dans l’application Surveys</span><span class="sxs-lookup"><span data-stu-id="19e14-132">Setting up Key Vault in the Surveys app</span></span>

<span data-ttu-id="19e14-133">Configuration requise :</span><span class="sxs-lookup"><span data-stu-id="19e14-133">Prerequisites:</span></span>

* <span data-ttu-id="19e14-134">Installez les [applets de commande Azure Resource Manager][azure-rm-cmdlets].</span><span class="sxs-lookup"><span data-stu-id="19e14-134">Install the [Azure Resource Manager Cmdlets][azure-rm-cmdlets].</span></span>
* <span data-ttu-id="19e14-135">Configurez l’application Surveys, comme indiqué dans [Exécuter l’application Surveys][readme].</span><span class="sxs-lookup"><span data-stu-id="19e14-135">Configure the Surveys application as described in [Run the Surveys application][readme].</span></span>

<span data-ttu-id="19e14-136">Procédure générale :</span><span class="sxs-lookup"><span data-stu-id="19e14-136">High-level steps:</span></span>

1. <span data-ttu-id="19e14-137">Configurez un utilisateur admin dans le client.</span><span class="sxs-lookup"><span data-stu-id="19e14-137">Set up an admin user in the tenant.</span></span>
2. <span data-ttu-id="19e14-138">Configurez un certificat client.</span><span class="sxs-lookup"><span data-stu-id="19e14-138">Set up a client certificate.</span></span>
3. <span data-ttu-id="19e14-139">Création d’un coffre de clés</span><span class="sxs-lookup"><span data-stu-id="19e14-139">Create a key vault.</span></span>
4. <span data-ttu-id="19e14-140">Ajoutez des paramètres de configuration à votre coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="19e14-140">Add configuration settings to your key vault.</span></span>
5. <span data-ttu-id="19e14-141">Supprimez les marques de commentaire du code qui active le coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="19e14-141">Uncomment the code that enables key vault.</span></span>
6. <span data-ttu-id="19e14-142">Mettez à jour les clés secrètes de l’utilisateur de l’application.</span><span class="sxs-lookup"><span data-stu-id="19e14-142">Update the application's user secrets.</span></span>

### <a name="set-up-an-admin-user"></a><span data-ttu-id="19e14-143">Configuration d’un utilisateur admin</span><span class="sxs-lookup"><span data-stu-id="19e14-143">Set up an admin user</span></span>

> [!NOTE]
> <span data-ttu-id="19e14-144">Pour créer un coffre de clés, vous devez utiliser un compte qui peut gérer votre abonnement Azure.</span><span class="sxs-lookup"><span data-stu-id="19e14-144">To create a key vault, you must use an account which can manage your Azure subscription.</span></span> <span data-ttu-id="19e14-145">De même, toute application que vous autorisez à lire dans le coffre de clés doit être inscrite dans le même locataire que ce compte.</span><span class="sxs-lookup"><span data-stu-id="19e14-145">Also, any application that you authorize to read from the key vault must be registered in the same tenant as that account.</span></span>

<span data-ttu-id="19e14-146">Dans cette étape, vous allez vous assurer que vous pouvez créer un coffre de clés lorsque vous êtes connecté en tant qu’utilisateur du client où l’application Surveys est inscrite.</span><span class="sxs-lookup"><span data-stu-id="19e14-146">In this step, you will make sure that you can create a key vault while signed in as a user from the tenant where the Surveys app is registered.</span></span>

<span data-ttu-id="19e14-147">Créez un utilisateur administrateur au sein du locataire Azure AD où l’application Surveys est inscrite.</span><span class="sxs-lookup"><span data-stu-id="19e14-147">Create an administrator user within the Azure AD tenant where the Surveys application is registered.</span></span>

1. <span data-ttu-id="19e14-148">Connectez-vous au [portail Azure][azure-portal].</span><span class="sxs-lookup"><span data-stu-id="19e14-148">Log into the [Azure portal][azure-portal].</span></span>
2. <span data-ttu-id="19e14-149">Sélectionnez le client Azure AD où votre application est inscrite.</span><span class="sxs-lookup"><span data-stu-id="19e14-149">Select the Azure AD tenant where your application is registered.</span></span>
3. <span data-ttu-id="19e14-150">Cliquez sur **Autres services** > **SÉCURITÉ + IDENTITÉ** > **Azure Active Directory** > **Utilisateurs et groupes**  > **Tous les utilisateurs**.</span><span class="sxs-lookup"><span data-stu-id="19e14-150">Click **More service** > **SECURITY + IDENTITY** > **Azure Active Directory** > **User and groups** > **All users**.</span></span>
4. <span data-ttu-id="19e14-151">Dans la partie supérieure du portail, cliquez sur **Nouvel utilisateur**.</span><span class="sxs-lookup"><span data-stu-id="19e14-151">At the top of the portal, click **New user**.</span></span>
5. <span data-ttu-id="19e14-152">Complétez les champs et assignez l’utilisateur au rôle d’annuaire **Administrateur général**.</span><span class="sxs-lookup"><span data-stu-id="19e14-152">Fill in the fields and assign the user to the **Global administrator** directory role.</span></span>
6. <span data-ttu-id="19e14-153">Cliquez sur **Créer**.</span><span class="sxs-lookup"><span data-stu-id="19e14-153">Click **Create**.</span></span>

![Utilisateur administrateur général](./images/running-the-app/global-admin-user.png)

<span data-ttu-id="19e14-155">Assignez maintenant cet utilisateur comme propriétaire d’abonnement.</span><span class="sxs-lookup"><span data-stu-id="19e14-155">Now assign this user as the subscription owner.</span></span>

1. <span data-ttu-id="19e14-156">Dans le menu Hub, sélectionnez **Abonnements**.</span><span class="sxs-lookup"><span data-stu-id="19e14-156">On the Hub menu, select **Subscriptions**.</span></span>

    ![Capture d’écran du hub du portail Azure](./images/running-the-app/subscriptions.png)

2. <span data-ttu-id="19e14-158">Sélectionnez l’abonnement auquel l’administrateur doit accéder.</span><span class="sxs-lookup"><span data-stu-id="19e14-158">Select the subscription that you want the administrator to access.</span></span>
3. <span data-ttu-id="19e14-159">Dans le panneau de l’abonnement, sélectionnez **Contrôle d’accès (IAM)**.</span><span class="sxs-lookup"><span data-stu-id="19e14-159">In the subscription blade, select **Access control (IAM)**.</span></span>
4. <span data-ttu-id="19e14-160">Cliquez sur **Add**.</span><span class="sxs-lookup"><span data-stu-id="19e14-160">Click **Add**.</span></span>
5. <span data-ttu-id="19e14-161">Sous **Rôle**, sélectionnez **Propriétaire**.</span><span class="sxs-lookup"><span data-stu-id="19e14-161">Under **Role**, select **Owner**.</span></span>
6. <span data-ttu-id="19e14-162">Tapez l’adresse e-mail de l’utilisateur à ajouter comme propriétaire.</span><span class="sxs-lookup"><span data-stu-id="19e14-162">Type the email address of the user you want to add as owner.</span></span>
7. <span data-ttu-id="19e14-163">Sélectionnez l’utilisateur et cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="19e14-163">Select the user and click **Save**.</span></span>

### <a name="set-up-a-client-certificate"></a><span data-ttu-id="19e14-164">Configuration d’un certificat client</span><span class="sxs-lookup"><span data-stu-id="19e14-164">Set up a client certificate</span></span>

1. <span data-ttu-id="19e14-165">Exécutez le script PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] comme suit :</span><span class="sxs-lookup"><span data-stu-id="19e14-165">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -Subject <<subject>>
    ```
    <span data-ttu-id="19e14-166">Pour le paramètre `Subject` , entrez un nom, comme « surveysapp ».</span><span class="sxs-lookup"><span data-stu-id="19e14-166">For the `Subject` parameter, enter any name, such as "surveysapp".</span></span> <span data-ttu-id="19e14-167">Le script génère un certificat auto-signé et le stocke dans le magasin de certificats « Utilisateur actuel/Personnel ».</span><span class="sxs-lookup"><span data-stu-id="19e14-167">The script generates a self-signed certificate and stores it in the "Current User/Personal" certificate store.</span></span> <span data-ttu-id="19e14-168">La sortie du script est un fragment JSON.</span><span class="sxs-lookup"><span data-stu-id="19e14-168">The output from the script is a JSON fragment.</span></span> <span data-ttu-id="19e14-169">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="19e14-169">Copy this value.</span></span>

2. <span data-ttu-id="19e14-170">Dans le [portail Azure][azure-portal], basculez vers le répertoire où est inscrite l’application Surveys en sélectionnant votre compte en haut à droite du portail.</span><span class="sxs-lookup"><span data-stu-id="19e14-170">In the [Azure portal][azure-portal], switch to the directory where the Surveys application is registered, by selecting your account in the top right corner of the portal.</span></span>

3. <span data-ttu-id="19e14-171">Sélectionnez **Azure Active Directory** > **Inscriptions des applications** > Surveys</span><span class="sxs-lookup"><span data-stu-id="19e14-171">Select **Azure Active Directory** > **App Registrations** > Surveys</span></span>

4. <span data-ttu-id="19e14-172">Cliquez sur **Manifeste**, puis sur **Modifier**.</span><span class="sxs-lookup"><span data-stu-id="19e14-172">Click **Manifest** and then **Edit**.</span></span>

5. <span data-ttu-id="19e14-173">Collez la sortie du script dans la propriété `keyCredentials` .</span><span class="sxs-lookup"><span data-stu-id="19e14-173">Paste the output from the script into the `keyCredentials` property.</span></span> <span data-ttu-id="19e14-174">Le résultat doit être semblable à ce qui suit :</span><span class="sxs-lookup"><span data-stu-id="19e14-174">It should look similar to the following:</span></span>

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

6. <span data-ttu-id="19e14-175">Cliquez sur **Enregistrer**.</span><span class="sxs-lookup"><span data-stu-id="19e14-175">Click **Save**.</span></span>

7. <span data-ttu-id="19e14-176">Répétez les étapes 3 à 6 pour ajouter le même fragment JSON au manifeste d’application de l’API web (Surveys.WebAPI).</span><span class="sxs-lookup"><span data-stu-id="19e14-176">Repeat steps 3-6 to add the same JSON fragment to the application manifest of the web API (Surveys.WebAPI).</span></span>

8. <span data-ttu-id="19e14-177">Dans la fenêtre PowerShell, exécutez la commande suivante pour obtenir l’empreinte numérique du certificat.</span><span class="sxs-lookup"><span data-stu-id="19e14-177">From the PowerShell window, run the following command to get the thumbprint of the certificate.</span></span>

    ```powershell
    certutil -store -user my [subject]
    ```

    <span data-ttu-id="19e14-178">Pour `[subject]`, utilisez la valeur que vous avez spécifiée pour Subject dans le script PowerShell.</span><span class="sxs-lookup"><span data-stu-id="19e14-178">For `[subject]`, use the value that you specified for Subject in the PowerShell script.</span></span> <span data-ttu-id="19e14-179">L’empreinte numérique est répertoriée sous « Cert Hash(sha1) ».</span><span class="sxs-lookup"><span data-stu-id="19e14-179">The thumbprint is listed under "Cert Hash(sha1)".</span></span> <span data-ttu-id="19e14-180">Copiez cette valeur.</span><span class="sxs-lookup"><span data-stu-id="19e14-180">Copy this value.</span></span> <span data-ttu-id="19e14-181">Vous utiliserez l’empreinte numérique ultérieurement.</span><span class="sxs-lookup"><span data-stu-id="19e14-181">You will use the thumbprint later.</span></span>

### <a name="create-a-key-vault"></a><span data-ttu-id="19e14-182">Création d’un coffre de clés</span><span class="sxs-lookup"><span data-stu-id="19e14-182">Create a key vault</span></span>

1. <span data-ttu-id="19e14-183">Exécutez le script PowerShell [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] comme suit :</span><span class="sxs-lookup"><span data-stu-id="19e14-183">Run the PowerShell script [/Scripts/Setup-KeyVault.ps1][Setup-KeyVault] as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ResourceGroupName <<resource group name>> -Location <<location>>
    ```

    <span data-ttu-id="19e14-184">Lorsque vous êtes invité à entrer vos informations d’identification, connectez-vous avec les informations de l’utilisateur Azure AD que vous avez créé précédemment.</span><span class="sxs-lookup"><span data-stu-id="19e14-184">When prompted for credentials, sign in as the Azure AD user that you created earlier.</span></span> <span data-ttu-id="19e14-185">Le script crée un groupe de ressources et un coffre de clés au sein de ce groupe de ressources.</span><span class="sxs-lookup"><span data-stu-id="19e14-185">The script creates a new resource group, and a new key vault within that resource group.</span></span>

2. <span data-ttu-id="19e14-186">Réexécutez Setup-KeyVault.ps1 comme suit :</span><span class="sxs-lookup"><span data-stu-id="19e14-186">Run Setup-KeyVault.ps1 again as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name>> -ApplicationIds @("<<Surveys app id>>", "<<Surveys.WebAPI app ID>>")
    ```

    <span data-ttu-id="19e14-187">Définissez les valeurs des paramètres suivantes :</span><span class="sxs-lookup"><span data-stu-id="19e14-187">Set the following parameter values:</span></span>

       * <span data-ttu-id="19e14-188">key vault name = Le nom que vous avez affecté au coffre de clés à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="19e14-188">key vault name = The name that you gave the key vault in the previous step.</span></span>
       * <span data-ttu-id="19e14-189">Surveys app ID = ID de l’application web Surveys.</span><span class="sxs-lookup"><span data-stu-id="19e14-189">Surveys app ID = The application ID for the Surveys web application.</span></span>
       * <span data-ttu-id="19e14-190">Surveys.WebApi app ID = ID de l’application Surveys.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="19e14-190">Surveys.WebApi app ID = The application ID for the Surveys.WebAPI application.</span></span>

    <span data-ttu-id="19e14-191">Exemple :</span><span class="sxs-lookup"><span data-stu-id="19e14-191">Example:</span></span>

    ```powershell
     .\Setup-KeyVault.ps1 -KeyVaultName tailspinkv -ApplicationIds @("f84df9d1-91cc-4603-b662-302db51f1031", "8871a4c2-2a23-4650-8b46-0625ff3928a6")
    ```

    <span data-ttu-id="19e14-192">Ce script autorise l’application web et l’API web à extraire les données secrètes de votre coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="19e14-192">This script authorizes the web app and web API to retrieve secrets from your key vault.</span></span> <span data-ttu-id="19e14-193">Pour plus d’informations, consultez [Prise en main du coffre de clés Azure](/azure/key-vault/key-vault-get-started/).</span><span class="sxs-lookup"><span data-stu-id="19e14-193">See [Get started with Azure Key Vault](/azure/key-vault/key-vault-get-started/) for more information.</span></span>

### <a name="add-configuration-settings-to-your-key-vault"></a><span data-ttu-id="19e14-194">Ajout de paramètres de configuration à votre coffre de clés</span><span class="sxs-lookup"><span data-stu-id="19e14-194">Add configuration settings to your key vault</span></span>

1. <span data-ttu-id="19e14-195">Exécutez Setup-KeyVault.ps1 comme suit :</span><span class="sxs-lookup"><span data-stu-id="19e14-195">Run Setup-KeyVault.ps1 as follows:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Redis--Configuration -KeyValue "<<Redis DNS name>>.redis.cache.windows.net,password=<<Redis access key>>,ssl=true"
    ```
    <span data-ttu-id="19e14-196">where</span><span class="sxs-lookup"><span data-stu-id="19e14-196">where</span></span>

   * <span data-ttu-id="19e14-197">key vault name = Le nom que vous avez affecté au coffre de clés à l’étape précédente.</span><span class="sxs-lookup"><span data-stu-id="19e14-197">key vault name = The name that you gave the key vault in the previous step.</span></span>
   * <span data-ttu-id="19e14-198">Redis DNS name = Le nom DNS de votre instance de cache Redis.</span><span class="sxs-lookup"><span data-stu-id="19e14-198">Redis DNS name = The DNS name of your Redis cache instance.</span></span>
   * <span data-ttu-id="19e14-199">Redis access key = la clé d’accès pour votre instance de cache Redis.</span><span class="sxs-lookup"><span data-stu-id="19e14-199">Redis access key = The access key for your Redis cache instance.</span></span>

2. <span data-ttu-id="19e14-200">À ce stade, il est judicieux de vérifier si vous avez stocké correctement les clés secrètes dans le coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="19e14-200">At this point, it's a good idea to test whether you successfully stored the secrets to key vault.</span></span> <span data-ttu-id="19e14-201">Exécutez la commande PowerShell suivante :</span><span class="sxs-lookup"><span data-stu-id="19e14-201">Run the following PowerShell command:</span></span>

    ```powershell
    Get-AzureKeyVaultSecret <<key vault name>> Redis--Configuration | Select-Object *
    ```

3. <span data-ttu-id="19e14-202">Réexécutez Setup-KeyVault.ps1 pour ajouter la chaîne de connexion de base de données :</span><span class="sxs-lookup"><span data-stu-id="19e14-202">Run Setup-KeyVault.ps1 again to add the database connection string:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName <<key vault name> -KeyName Data--SurveysConnectionString -KeyValue <<DB connection string>> -ConfigName "Data:SurveysConnectionString"
    ```

    <span data-ttu-id="19e14-203">où `<<DB connection string>>` est la valeur de la chaîne de connexion de base de données.</span><span class="sxs-lookup"><span data-stu-id="19e14-203">where `<<DB connection string>>` is the value of the database connection string.</span></span>

    <span data-ttu-id="19e14-204">Pour effectuer un test avec la base de données locale, copiez la chaîne de connexion à partir du fichier Tailspin.Surveys.Web/appsettings.json.</span><span class="sxs-lookup"><span data-stu-id="19e14-204">For testing with the local database, copy the connection string from the Tailspin.Surveys.Web/appsettings.json file.</span></span> <span data-ttu-id="19e14-205">Si vous faites cela, veillez à remplacer la double barre oblique inverse (« \\\\ ») par une simple barre oblique inverse.</span><span class="sxs-lookup"><span data-stu-id="19e14-205">If you do that, make sure to change the double backslash ('\\\\') into a single backslash.</span></span> <span data-ttu-id="19e14-206">La double barre oblique inverse est un caractère d’échappement dans le fichier JSON.</span><span class="sxs-lookup"><span data-stu-id="19e14-206">The double backslash is an escape character in the JSON file.</span></span>

    <span data-ttu-id="19e14-207">Exemple :</span><span class="sxs-lookup"><span data-stu-id="19e14-207">Example:</span></span>

    ```powershell
    .\Setup-KeyVault.ps1 -KeyVaultName mykeyvault -KeyName Data--SurveysConnectionString -KeyValue "Server=(localdb)\MSSQLLocalDB;Database=Tailspin.SurveysDB;Trusted_Connection=True;MultipleActiveResultSets=true"
    ```

### <a name="uncomment-the-code-that-enables-key-vault"></a><span data-ttu-id="19e14-208">Suppression des marques de commentaire du code qui active le coffre de clés</span><span class="sxs-lookup"><span data-stu-id="19e14-208">Uncomment the code that enables Key Vault</span></span>

1. <span data-ttu-id="19e14-209">Ouvrez la solution Tailspin.Surveys.</span><span class="sxs-lookup"><span data-stu-id="19e14-209">Open the Tailspin.Surveys solution.</span></span>
2. <span data-ttu-id="19e14-210">Dans Tailspin.Surveys.Web/Startup.cs, localisez le bloc de code suivant et supprimez les commentaires.</span><span class="sxs-lookup"><span data-stu-id="19e14-210">In Tailspin.Surveys.Web/Startup.cs, locate the following code block and uncomment it.</span></span>

    ```csharp
    //var config = builder.Build();
    //builder.AddAzureKeyVault(
    //    $"https://{config["KeyVault:Name"]}.vault.azure.net/",
    //    config["AzureAd:ClientId"],
    //    config["AzureAd:ClientSecret"]);
    ```
3. <span data-ttu-id="19e14-211">Dans Tailspin.Surveys.Web/Startup.cs, localisez le code qui inscrit `ICredentialService`.</span><span class="sxs-lookup"><span data-stu-id="19e14-211">In Tailspin.Surveys.Web/Startup.cs, locate the code that registers the `ICredentialService`.</span></span> <span data-ttu-id="19e14-212">Supprimez les commentaires de la ligne qui utilise `CertificateCredentialService`, puis commentez la ligne qui utilise `ClientCredentialService` :</span><span class="sxs-lookup"><span data-stu-id="19e14-212">Uncomment the line that uses `CertificateCredentialService`, and comment out the line that uses `ClientCredentialService`:</span></span>

    ```csharp
    // Uncomment this:
    services.AddSingleton<ICredentialService, CertificateCredentialService>();
    // Comment out this:
    //services.AddSingleton<ICredentialService, ClientCredentialService>();
    ```

    <span data-ttu-id="19e14-213">Cette modification permet à l’application web d’utiliser l’[Assertion de client][client-assertion] pour obtenir des jetons d’accès OAuth.</span><span class="sxs-lookup"><span data-stu-id="19e14-213">This change enables the web app to use [Client assertion][client-assertion] to get OAuth access tokens.</span></span> <span data-ttu-id="19e14-214">Avec l’assertion de client, vous n’avez pas besoin de secret client OAuth.</span><span class="sxs-lookup"><span data-stu-id="19e14-214">With client assertion, you don't need an OAuth client secret.</span></span> <span data-ttu-id="19e14-215">Vous pouvez aussi stocker le secret client dans le coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="19e14-215">Alternatively, you could store the client secret in key vault.</span></span> <span data-ttu-id="19e14-216">Cependant, le coffre de clés et l’assertion de client utilisent tous deux un certificat client. Donc, si vous activez le coffre de clés, il est recommandé d’activer aussi l’assertion de client.</span><span class="sxs-lookup"><span data-stu-id="19e14-216">However, key vault and client assertion both use a client certificate, so if you enable key vault, it's a good practice to enable client assertion as well.</span></span>

### <a name="update-the-user-secrets"></a><span data-ttu-id="19e14-217">Mise à jour des données secrètes de l’utilisateur</span><span class="sxs-lookup"><span data-stu-id="19e14-217">Update the user secrets</span></span>

<span data-ttu-id="19e14-218">Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet Tailspin.Surveys.Web, puis sélectionnez **Gérer les données secrètes de l’utilisateur**.</span><span class="sxs-lookup"><span data-stu-id="19e14-218">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span> <span data-ttu-id="19e14-219">Dans le fichier secrets.json, supprimez le script JSON existant et collez les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="19e14-219">In the secrets.json file, delete the existing JSON and paste in the following:</span></span>

```json
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

<span data-ttu-id="19e14-220">Remplacez les entrées entre [crochets] par les valeurs correctes.</span><span class="sxs-lookup"><span data-stu-id="19e14-220">Replace the entries in [square brackets] with the correct values.</span></span>

* <span data-ttu-id="19e14-221">`AzureAd:ClientId`: ID client de l’application Surveys.</span><span class="sxs-lookup"><span data-stu-id="19e14-221">`AzureAd:ClientId`: The client ID of the Surveys app.</span></span>
* <span data-ttu-id="19e14-222">`AzureAd:ClientSecret`: clé que vous avez générée au moment d’inscrire l’application Surveys dans Azure AD.</span><span class="sxs-lookup"><span data-stu-id="19e14-222">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
* <span data-ttu-id="19e14-223">`AzureAd:WebApiResourceId`: URI d’ID de l’application que vous avez spécifié quand vous avez créé l’application Surveys.WebAPI dans Azure AD.</span><span class="sxs-lookup"><span data-stu-id="19e14-223">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span>
* <span data-ttu-id="19e14-224">`Asymmetric:CertificateThumbprint`: Empreinte numérique de certificat obtenue quand vous avez créé le certificat client.</span><span class="sxs-lookup"><span data-stu-id="19e14-224">`Asymmetric:CertificateThumbprint`: The certificate thumbprint that you got previously, when you created the client certificate.</span></span>
* <span data-ttu-id="19e14-225">`KeyVault:Name`: Nom de votre coffre de clés.</span><span class="sxs-lookup"><span data-stu-id="19e14-225">`KeyVault:Name`: The name of your key vault.</span></span>

> [!NOTE]
> <span data-ttu-id="19e14-226">`Asymmetric:ValidationRequired` a la valeur false, car le certificat que vous avez créé précédemment n’a pas été signé par une autorité de certification racine.</span><span class="sxs-lookup"><span data-stu-id="19e14-226">`Asymmetric:ValidationRequired` is false because the certificate that you created previously was not signed by a root certificate authority (CA).</span></span> <span data-ttu-id="19e14-227">En production, utilisez un certificat signé par une autorité de certification racine, puis affectez à `ValidationRequired` la valeur true.</span><span class="sxs-lookup"><span data-stu-id="19e14-227">In production, use a certificate that is signed by a root CA and set `ValidationRequired` to true.</span></span>

<span data-ttu-id="19e14-228">Enregistrez le fichier secrets.json mis à jour.</span><span class="sxs-lookup"><span data-stu-id="19e14-228">Save the updated secrets.json file.</span></span>

<span data-ttu-id="19e14-229">Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet Tailspin.Surveys.WebApi, puis sélectionnez **Gérer les données secrètes de l’utilisateur**.</span><span class="sxs-lookup"><span data-stu-id="19e14-229">Next, in Solution Explorer, right-click the Tailspin.Surveys.WebApi project and select **Manage User Secrets**.</span></span> <span data-ttu-id="19e14-230">Supprimez le script JSON existant et collez les éléments suivants :</span><span class="sxs-lookup"><span data-stu-id="19e14-230">Delete the existing JSON and paste in the following:</span></span>

```json
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

<span data-ttu-id="19e14-231">Remplacez les entrées entre [crochets] et enregistrez le fichier secrets.json.</span><span class="sxs-lookup"><span data-stu-id="19e14-231">Replace the entries in [square brackets] and save the secrets.json file.</span></span>

> [!NOTE]
> <span data-ttu-id="19e14-232">Pour l’API web, veillez à utiliser l’ID client de l’application Surveys.WebAPI et non de l’application Surveys.</span><span class="sxs-lookup"><span data-stu-id="19e14-232">For the web API, make sure to use the client ID for the Surveys.WebAPI application, not the Surveys application.</span></span>

<span data-ttu-id="19e14-233">[**Suivant**][adfs]</span><span class="sxs-lookup"><span data-stu-id="19e14-233">[**Next**][adfs]</span></span>

<!-- links -->

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
[user-secrets]: /aspnet/core/security/app-secrets
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
