---
title: Intégrer un environnement Active Directory local à Azure
titleSuffix: Azure Reference Architectures
description: Comparatif des différentes architectures de référence pour intégrer un environnement Active Directory local à Azure.
ms.date: 07/02/2018
ms.custom: seodec18
ms.openlocfilehash: 905dedda6de1a107f55b2f7651441780a685aea7
ms.sourcegitcommit: 88a68c7e9b6b772172b7faa4b9fd9c061a9f7e9d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2018
ms.locfileid: "53119861"
---
# <a name="choose-a-solution-for-integrating-on-premises-active-directory-with-azure"></a><span data-ttu-id="6123b-103">Choisir une solution pour intégrer l’environnement Active Directory local à Azure</span><span class="sxs-lookup"><span data-stu-id="6123b-103">Choose a solution for integrating on-premises Active Directory with Azure</span></span>

<span data-ttu-id="6123b-104">Cet article compare les options disponibles pour intégrer votre environnement Active Directory (AD) local avec un réseau Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-104">This article compares options for integrating your on-premises Active Directory (AD) environment with an Azure network.</span></span> <span data-ttu-id="6123b-105">Pour chaque option, une architecture de référence plus détaillée est disponible.</span><span class="sxs-lookup"><span data-stu-id="6123b-105">For each option, a more detailed reference architecture is available.</span></span>

<span data-ttu-id="6123b-106">De nombreuses organisations utilisent les services Active Directory Domain Services (AD DS) pour authentifier les identités associées aux utilisateurs, aux ordinateurs, aux applications ou aux autres ressources incluses dans un périmètre de sécurité donné.</span><span class="sxs-lookup"><span data-stu-id="6123b-106">Many organizations use Active Directory Domain Services (AD DS) to authenticate identities associated with users, computers, applications, or other resources that are included in a security boundary.</span></span> <span data-ttu-id="6123b-107">Les services de répertoire et d’identité sont généralement hébergés localement, mais si votre application est hébergée à la fois localement et dans Azure, il peut y avoir un certain temps de latence lors de l’envoi de requêtes d’authentification à partir d’Azure vers l’environnement local.</span><span class="sxs-lookup"><span data-stu-id="6123b-107">Directory and identity services are typically hosted on-premises, but if your application is hosted partly on-premises and partly in Azure, there may be latency sending authentication requests from Azure back to on-premises.</span></span> <span data-ttu-id="6123b-108">L’implémentation des services de répertoire et d’identité dans Azure peut réduire cette latence.</span><span class="sxs-lookup"><span data-stu-id="6123b-108">Implementing directory and identity services in Azure can reduce this latency.</span></span>

<span data-ttu-id="6123b-109">Azure offre deux solutions pour implémenter les services de répertoire et d’identité dans Azure :</span><span class="sxs-lookup"><span data-stu-id="6123b-109">Azure provides two solutions for implementing directory and identity services in Azure:</span></span>

- <span data-ttu-id="6123b-110">Utilisez [Azure AD][azure-active-directory] pour créer un domaine Active Directory dans le cloud et le connecter à votre domaine Active Directory local.</span><span class="sxs-lookup"><span data-stu-id="6123b-110">Use [Azure AD][azure-active-directory] to create an Active Directory domain in the cloud and connect it to your on-premises Active Directory domain.</span></span> <span data-ttu-id="6123b-111">[Azure AD Connect][azure-ad-connect] intègre vos répertoires locaux à Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6123b-111">[Azure AD Connect][azure-ad-connect] integrates your on-premises directories with Azure AD.</span></span>

- <span data-ttu-id="6123b-112">Étendez votre infrastructure Active Directory locale dans Azure en déployant une machine virtuelle dans Azure qui exécute les services AD DS en tant que contrôleur de domaine.</span><span class="sxs-lookup"><span data-stu-id="6123b-112">Extend your existing on-premises Active Directory infrastructure to Azure, by deploying a VM in Azure that runs AD DS as a domain controller.</span></span> <span data-ttu-id="6123b-113">Cette architecture est couramment utilisée quand le réseau local et le réseau virtuel Azure (VNet) sont connectés par une connexion VPN ou ExpressRoute.</span><span class="sxs-lookup"><span data-stu-id="6123b-113">This architecture is more common when the on-premises network and the Azure virtual network (VNet) are connected by a VPN or ExpressRoute connection.</span></span> <span data-ttu-id="6123b-114">Il existe plusieurs variantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="6123b-114">Several variations of this architecture are possible:</span></span>

  - <span data-ttu-id="6123b-115">Créez un domaine dans Azure et associez-le à votre forêt AD locale.</span><span class="sxs-lookup"><span data-stu-id="6123b-115">Create a domain in Azure and join it to your on-premises AD forest.</span></span>
  - <span data-ttu-id="6123b-116">Créez une forêt distincte dans Azure qui est approuvée par des domaines de votre forêt locale.</span><span class="sxs-lookup"><span data-stu-id="6123b-116">Create a separate forest in Azure that is trusted by domains in your on-premises forest.</span></span>
  - <span data-ttu-id="6123b-117">Répliquez un déploiement Active Directory Federation Services (AD FS) dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-117">Replicate an Active Directory Federation Services (AD FS) deployment to Azure.</span></span>

<span data-ttu-id="6123b-118">Les sections suivantes décrivent chacune de ces options plus en détails.</span><span class="sxs-lookup"><span data-stu-id="6123b-118">The next sections describe each of these options in more detail.</span></span>

## <a name="integrate-your-on-premises-domains-with-azure-ad"></a><span data-ttu-id="6123b-119">Intégrer vos domaines locaux à Azure AD</span><span class="sxs-lookup"><span data-stu-id="6123b-119">Integrate your on-premises domains with Azure AD</span></span>

<span data-ttu-id="6123b-120">Utilisez Azure Active Directory (Azure AD) pour créer un domaine dans Azure et l’associer à un domaine AD local.</span><span class="sxs-lookup"><span data-stu-id="6123b-120">Use Azure Active Directory (Azure AD) to create a domain in Azure and link it to an on-premises AD domain.</span></span>

<span data-ttu-id="6123b-121">Le répertoire Azure AD n’est pas une extension d’un répertoire local.</span><span class="sxs-lookup"><span data-stu-id="6123b-121">The Azure AD directory is not an extension of an on-premises directory.</span></span> <span data-ttu-id="6123b-122">Il s’agit en réalité d’une copie qui contient les mêmes objets et identités.</span><span class="sxs-lookup"><span data-stu-id="6123b-122">Rather, it's a copy that contains the same objects and identities.</span></span> <span data-ttu-id="6123b-123">Les modifications apportées à ces éléments locaux sont copiées vers Azure AD, mais les modifications apportées dans Azure AD ne sont pas répliquées sur le domaine local.</span><span class="sxs-lookup"><span data-stu-id="6123b-123">Changes made to these items on-premises are copied to Azure AD, but changes made in Azure AD are not replicated back to the on-premises domain.</span></span>

<span data-ttu-id="6123b-124">Vous pouvez également utiliser Azure AD sans utiliser un répertoire local.</span><span class="sxs-lookup"><span data-stu-id="6123b-124">You can also use Azure AD without using an on-premises directory.</span></span> <span data-ttu-id="6123b-125">Dans ce cas, Azure AD agit comme la source principale de toutes les informations d’identité, au lieu d’obtenir des données répliquées d’un répertoire local.</span><span class="sxs-lookup"><span data-stu-id="6123b-125">In this case, Azure AD acts as the primary source of all identity information, rather than containing data replicated from an on-premises directory.</span></span>

<span data-ttu-id="6123b-126">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="6123b-126">**Benefits**</span></span>

- <span data-ttu-id="6123b-127">Vous n’avez pas besoin de gérer une infrastructure AD dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="6123b-127">You don't need to maintain an AD infrastructure in the cloud.</span></span> <span data-ttu-id="6123b-128">Azure AD est entièrement géré et mis à jour par Microsoft.</span><span class="sxs-lookup"><span data-stu-id="6123b-128">Azure AD is entirely managed and maintained by Microsoft.</span></span>
- <span data-ttu-id="6123b-129">Azure AD fournit les mêmes informations d’identité que celles disponibles localement.</span><span class="sxs-lookup"><span data-stu-id="6123b-129">Azure AD provides the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="6123b-130">L’authentification peut s’effectuer dans Azure, ce qui évite aux utilisateurs et applications externes d’avoir à contacter le domaine local.</span><span class="sxs-lookup"><span data-stu-id="6123b-130">Authentication can happen in Azure, reducing the need for external applications and users to contact the on-premises domain.</span></span>

<span data-ttu-id="6123b-131">**Défis**</span><span class="sxs-lookup"><span data-stu-id="6123b-131">**Challenges**</span></span>

- <span data-ttu-id="6123b-132">Les services d’identité sont réservés aux utilisateurs et aux groupes.</span><span class="sxs-lookup"><span data-stu-id="6123b-132">Identity services are limited to users and groups.</span></span> <span data-ttu-id="6123b-133">Aucune authentification de comptes de service et d’ordinateur n’est possible.</span><span class="sxs-lookup"><span data-stu-id="6123b-133">There is no ability to authenticate service and computer accounts.</span></span>
- <span data-ttu-id="6123b-134">Vous devez configurer la connectivité à votre domaine local pour assurer la synchronisation du répertoire Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6123b-134">You must configure connectivity with your on-premises domain to keep the Azure AD directory synchronized.</span></span> 
- <span data-ttu-id="6123b-135">Les applications devront peut-être être réécrites pour activer l’authentification via Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6123b-135">Applications may need to be rewritten to enable authentication through Azure AD.</span></span>

<span data-ttu-id="6123b-136">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="6123b-136">**Reference architecture**</span></span>

- <span data-ttu-id="6123b-137">[Intégrer des domaines Active Directory locaux avec Azure Active Directory][aad]</span><span class="sxs-lookup"><span data-stu-id="6123b-137">[Integrate on-premises Active Directory domains with Azure Active Directory][aad]</span></span>

## <a name="ad-ds-in-azure-joined-to-an-on-premises-forest"></a><span data-ttu-id="6123b-138">Services AD DS dans Azure associés à une forêt locale</span><span class="sxs-lookup"><span data-stu-id="6123b-138">AD DS in Azure joined to an on-premises forest</span></span>

<span data-ttu-id="6123b-139">Déployer des serveurs AD Domain Services (AD DS) dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-139">Deploy AD Domain Services (AD DS) servers to Azure.</span></span> <span data-ttu-id="6123b-140">Créez un domaine dans Azure et associez-le à votre forêt AD locale.</span><span class="sxs-lookup"><span data-stu-id="6123b-140">Create a domain in Azure and join it to your on-premises AD forest.</span></span> 

<span data-ttu-id="6123b-141">Envisagez cette option si vous avez besoin d’utiliser des fonctionnalités AD DS qui ne sont pas actuellement disponibles dans Azure AD.</span><span class="sxs-lookup"><span data-stu-id="6123b-141">Consider this option if you need to use AD DS features that are not currently implemented by Azure AD.</span></span> 

<span data-ttu-id="6123b-142">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="6123b-142">**Benefits**</span></span>

- <span data-ttu-id="6123b-143">Cette méthode fournit les mêmes informations d’identité que celles disponibles localement.</span><span class="sxs-lookup"><span data-stu-id="6123b-143">Provides access to the same identity information that is available on-premises.</span></span>
- <span data-ttu-id="6123b-144">Vous pouvez authentifier les comptes d’utilisateur, de service et d’ordinateur localement et dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-144">You can authenticate user, service, and computer accounts on-premises and in Azure.</span></span>
- <span data-ttu-id="6123b-145">Vous n’avez pas besoin de gérer une autre forêt Active Directory.</span><span class="sxs-lookup"><span data-stu-id="6123b-145">You don't need to manage a separate AD forest.</span></span> <span data-ttu-id="6123b-146">Le domaine dans Azure peut appartenir à la forêt locale.</span><span class="sxs-lookup"><span data-stu-id="6123b-146">The domain in Azure can belong to the on-premises forest.</span></span>
- <span data-ttu-id="6123b-147">Vous pouvez appliquer la stratégie de groupe définie par les objets de stratégie de groupe locale au domaine dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-147">You can apply group policy defined by on-premises Group Policy Objects to the domain in Azure.</span></span>

<span data-ttu-id="6123b-148">**Défis**</span><span class="sxs-lookup"><span data-stu-id="6123b-148">**Challenges**</span></span>

- <span data-ttu-id="6123b-149">Vous devez déployer et gérer vos propres domaine et serveurs AD DS dans le cloud.</span><span class="sxs-lookup"><span data-stu-id="6123b-149">You must deploy and manage your own AD DS servers and domain in the cloud.</span></span>
- <span data-ttu-id="6123b-150">Il peut y avoir une certaine latence au niveau de la synchronisation entre les serveurs de domaine dans le cloud et les serveurs exécutés localement.</span><span class="sxs-lookup"><span data-stu-id="6123b-150">There may be some synchronization latency between the domain servers in the cloud and the servers running on-premises.</span></span>

<span data-ttu-id="6123b-151">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="6123b-151">**Reference architecture**</span></span>

- <span data-ttu-id="6123b-152">[Étendre Active Directory Domain Services (AD DS) à Azure][ad-ds]</span><span class="sxs-lookup"><span data-stu-id="6123b-152">[Extend Active Directory Domain Services (AD DS) to Azure][ad-ds]</span></span>

## <a name="ad-ds-in-azure-with-a-separate-forest"></a><span data-ttu-id="6123b-153">Services AD DS dans Azure avec une forêt distincte</span><span class="sxs-lookup"><span data-stu-id="6123b-153">AD DS in Azure with a separate forest</span></span>

<span data-ttu-id="6123b-154">Déployez des serveurs AD Domain Services (AD DS) dans Azure, mais créez une [forêt][ad-forest-defn] Active Directory différente de la forêt locale.</span><span class="sxs-lookup"><span data-stu-id="6123b-154">Deploy AD Domain Services (AD DS) servers to Azure, but create a separate Active Directory [forest][ad-forest-defn] that is separate from the on-premises forest.</span></span> <span data-ttu-id="6123b-155">Cette forêt est approuvée par les domaines de votre forêt locale.</span><span class="sxs-lookup"><span data-stu-id="6123b-155">This forest is trusted by domains in your on-premises forest.</span></span>

<span data-ttu-id="6123b-156">Parmi les utilisations courantes de cette architecture figurent la conservation d’une séparation de sécurité pour les objets et les identités contenus dans le cloud et la migration de domaines individuels depuis l’environnement local vers le cloud.</span><span class="sxs-lookup"><span data-stu-id="6123b-156">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="6123b-157">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="6123b-157">**Benefits**</span></span>

- <span data-ttu-id="6123b-158">Vous pouvez implémenter des identités locales, ainsi que des identités Azure distinctes.</span><span class="sxs-lookup"><span data-stu-id="6123b-158">You can implement on-premises identities and separate Azure-only identities.</span></span>
- <span data-ttu-id="6123b-159">Vous n’avez pas besoin de répliquer les données de la forêt Active Directory locale vers Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-159">You don't need to replicate from the on-premises AD forest to Azure.</span></span>

<span data-ttu-id="6123b-160">**Défis**</span><span class="sxs-lookup"><span data-stu-id="6123b-160">**Challenges**</span></span>

- <span data-ttu-id="6123b-161">Dans Azure, l’authentification des identités locales nécessite des tronçons réseaux supplémentaires vers les serveurs Active Directory locaux.</span><span class="sxs-lookup"><span data-stu-id="6123b-161">Authentication within Azure for on-premises identities requires extra network hops to the on-premises AD servers.</span></span>
- <span data-ttu-id="6123b-162">Vous devez déployer vos propres forêt et serveurs AD DS dans le cloud et établir les relations d’approbation appropriées entre les forêts.</span><span class="sxs-lookup"><span data-stu-id="6123b-162">You must deploy your own AD DS servers and forest in the cloud, and establish the appropriate trust relationships between forests.</span></span>

<span data-ttu-id="6123b-163">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="6123b-163">**Reference architecture**</span></span>

- <span data-ttu-id="6123b-164">[Créer une forêt de ressources Active Directory Domain Services (AD DS) dans Azure][ad-ds-forest]</span><span class="sxs-lookup"><span data-stu-id="6123b-164">[Create an Active Directory Domain Services (AD DS) resource forest in Azure][ad-ds-forest]</span></span>

## <a name="extend-ad-fs-to-azure"></a><span data-ttu-id="6123b-165">Étendre AD FS à Azure</span><span class="sxs-lookup"><span data-stu-id="6123b-165">Extend AD FS to Azure</span></span>

<span data-ttu-id="6123b-166">Répliquez un déploiement Active Directory Federation Services (AD FS) dans Azure afin de permettre l’authentification et l’autorisation fédérées des composants en cours d’exécution dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-166">Replicate an Active Directory Federation Services (AD FS) deployment to Azure, to perform federated authentication and authorization for components running in Azure.</span></span> 

<span data-ttu-id="6123b-167">Utilisations courantes de cette architecture :</span><span class="sxs-lookup"><span data-stu-id="6123b-167">Typical uses for this architecture:</span></span>

- <span data-ttu-id="6123b-168">Authentifier et autoriser les utilisateurs des organisations partenaires.</span><span class="sxs-lookup"><span data-stu-id="6123b-168">Authenticate and authorize users from partner organizations.</span></span>
- <span data-ttu-id="6123b-169">Permettre aux utilisateurs de s’authentifier à partir de navigateurs web s’exécutant en dehors du pare-feu de l’organisation.</span><span class="sxs-lookup"><span data-stu-id="6123b-169">Allow users to authenticate from web browsers running outside of the organizational firewall.</span></span>
- <span data-ttu-id="6123b-170">Permettre aux utilisateurs de se connecter à partir d’appareils externes autorisés tels que des appareils mobiles.</span><span class="sxs-lookup"><span data-stu-id="6123b-170">Allow users to connect from authorized external devices such as mobile devices.</span></span> 

<span data-ttu-id="6123b-171">**Avantages**</span><span class="sxs-lookup"><span data-stu-id="6123b-171">**Benefits**</span></span>

- <span data-ttu-id="6123b-172">Vous pouvez utiliser des applications prenant en charge les revendications.</span><span class="sxs-lookup"><span data-stu-id="6123b-172">You can leverage claims-aware applications.</span></span>
- <span data-ttu-id="6123b-173">Vous pouvez approuver des partenaires externes pour l’authentification.</span><span class="sxs-lookup"><span data-stu-id="6123b-173">Provides the ability to trust external partners for authentication.</span></span>
- <span data-ttu-id="6123b-174">Cette méthode est compatible avec un large éventail de protocoles d’authentification.</span><span class="sxs-lookup"><span data-stu-id="6123b-174">Compatibility with large set of authentication protocols.</span></span>

<span data-ttu-id="6123b-175">**Défis**</span><span class="sxs-lookup"><span data-stu-id="6123b-175">**Challenges**</span></span>

- <span data-ttu-id="6123b-176">Vous devez déployer vos propres serveurs proxy d’application web AD FS, AD DS et AD FS dans Azure.</span><span class="sxs-lookup"><span data-stu-id="6123b-176">You must deploy your own AD DS, AD FS, and AD FS Web Application Proxy servers in Azure.</span></span>
- <span data-ttu-id="6123b-177">Cette architecture peut être difficile à configurer.</span><span class="sxs-lookup"><span data-stu-id="6123b-177">This architecture can be complex to configure.</span></span>

<span data-ttu-id="6123b-178">**Architecture de référence**</span><span class="sxs-lookup"><span data-stu-id="6123b-178">**Reference architecture**</span></span>

- <span data-ttu-id="6123b-179">[Étendre les services de fédération Active Directory (AD FS) dans Azure][adfs]</span><span class="sxs-lookup"><span data-stu-id="6123b-179">[Extend Active Directory Federation Services (AD FS) to Azure][adfs]</span></span>

<!-- links -->

[aad]: ./azure-ad.md
[ad-ds]: ./adds-extend-domain.md
[ad-ds-forest]: ./adds-forest.md
[ad-forest-defn]: /windows/desktop/AD/forests
[adfs]: ./adfs.md

[azure-active-directory]: /azure/active-directory-domain-services/active-directory-ds-overview
[azure-ad-connect]: /azure/active-directory/hybrid/whatis-hybrid-identity
