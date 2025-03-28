---
title: "Using Graph API for Entra Connect Sync version check"
seoTitle: "Cloud Synchronization Client Version"
seoDescription: "Fetching synchronizationClientVersion by Graph API"
datePublished: Fri Mar 28 2025 18:17:02 GMT+0000 (Coordinated Universal Time)
cuid: cm8t3twm5000b09lb566b4wts
slug: using-graph-api-for-entra-connect-sync-version-check
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743185251734/cedaa8e7-e3ec-47ab-ac34-876056496cde.png
tags: powershell, hybrid-cloud, identity-management, graph-api, entra-id, entra-connect-sync

---

forgive me for not starting by what is a hybrid environment regarding Active Directory Domain Services (AD DS) and Entra ID. Let’s break it down to this:

# Intro

If you want your objects in sync in both directories you can have to options:

1. Entra Connect Sync (also known als Azure AD Connect in the good ol’ days)
    
2. Entra Cloud Sync
    

Today’s post is about the Connect Sync service. Long story short: The engine enables synchronization between on-premises Active Directory, Entra ID, and other third-party systems, ensuring consistency across the entire identity ecosystem.

Key components include the **Connector**, which links to source directories and manages data flow, and the **Synchronization Engine**, which handles the logic of mapping and transforming identity data between systems. The **Agent** is a lightweight service installed on the server, responsible for executing synchronization tasks:

![Technical Concepts](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/media/how-to-connect-sync-technical-concepts/scenario.png align="left")

In case you want to learn more about it, I recommend:

* [Microsoft Entra Connect Sync: Technical concepts - Microsoft Entra ID | Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-sync-technical-concepts)
    
* [Microsoft Entra Connect Sync: Understanding the architecture - Azure - Microsoft Entra ID | Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/concept-azure-ad-connect-sync-architecture)
    

# Issue

Back to business - the sync engine has to be updated. [Auto Upgrade](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-install-automatic-upgrade#auto-upgrade-eligibility) is eligible in some cases:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743177715777/96359690-4d29-4602-8e3d-4975d6503aa7.png align="center")

Ok, great but this post is because if a “[**Breaking Change on Entra Connect Sync**](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/reference-connect-version-history#breaking-change-on-entra-connect-sync)”:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743177979057/dc419631-b7eb-4a32-826e-6d90c738701c.png align="center")

The latest version at the moment I am typing here is [**2.4.129.0**](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/reference-connect-version-history#241290) and you have to have at least 2.4.18 (for commercial clouds) or 2.4.21.0 (for non-commercial clouds).  
If you do not upgrade by the deadline of April 7,2025 the [impacts](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/harden-update-ad-fs-pingfederate?wt.mc_id=MVP_353010#expected-impacts) are:

* All authentication requests to Microsoft Entra ID on the Microsoft Entra Connect wizard will fail.
    
* Configuration of Active Directory Federation Services (ADFS) scenarios through Microsoft Entra Connect wizard won't work.
    
* Configuration of PingFederate scenarios through the Microsoft Entra Connect wizard won't work.
    

But the good thing about that: The sync service will continue doing it’s job! ✅

# Using Graph API

In case you want to automate the reporting on the Connect servers, where the agent is installed you should use Graph API. Back in the days I used the MSOL PowerShell module for that, but now this is gone, because of it’s [deprecation](https://techcommunity.microsoft.com/blog/microsoft-entra-blog/important-update-deprecation-of-azure-ad-powershell-and-msonline-powershell-modu/4094536). Since then I struggled finding the attributes which I was able to fetch back then by running:

```powershell
Get-MsolCompanyInformation | select DisplayName,InitialDomain,DirSyncClientMachineName,DirSyncClientVersion,DirSyncServiceAccount,DirectorySynchronizationEnabled, DirectorySynchronizationStatus, LastDirSyncTime, LastPasswordSyncTime 
```

Especially the **DirSyncClientVersion** is the attribute I am looking for. Mapping old cmdlets to new ones can be done via [Find Azure AD and MSOnline cmdlets in Microsoft Graph PowerShell | Microsoft Learn](https://learn.microsoft.com/en-us/powershell/microsoftgraph/azuread-msoline-cmdlet-map?view=graph-powershell-1.0&pivots=msonline). But both the [Get-MgOrganization](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.identity.directorymanagement/get-mgorganization) and the [Get-MgBetaOrganization](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.beta.identity.directorymanagement/get-mgbetaorganization?view=graph-powershell-beta) did not get me these insights. So I had a look at the native Graph API and found **directory/onPremisesSynchronization.**

For quick access to Graph API, you should use [Graph Explorer](https://aka.ms/ge) or Postman. So I called [https://graph.microsoft.com/beta/directory/onPremisesSynchronization](https://graph.microsoft.com/beta/directory/onPremisesSynchronization) to get some insights:

```http
GET https://graph.microsoft.com/v1.0/directory/onPremisesSynchronization
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743182544805/faa7d355-5af5-4ec3-b5d4-9d5604f5d890.png align="center")

Sadly, I did only get some configuration features. I just tried then the **beta** version of the API by calling:

```http
GET https://graph.microsoft.com/beta/directory/onPremisesSynchronization
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743182587531/769b89b1-4620-47b7-9d38-4adb09399285.png align="center")

And there it is: **synchronizationClientVersion‼️**

While writing this post, I checked the [Get-MgBetaDirectoryOnPremiseSynchronization](https://learn.microsoft.com/en-us/powershell/module/microsoft.graph.beta.identity.directorymanagement/get-mgbetadirectoryonpremisesynchronization?view=graph-powershell-beta). And here we go:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743183565741/41d6cb10-3a19-4e28-a49f-f26bd27d0ab1.png align="center")

Nevertheless: I hope, if someone looks for that attribute he will find it here 😁

Last but not least… [Consider moving to Microsoft Entra Cloud Sync](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/harden-update-ad-fs-pingfederate#consider-moving-to-microsoft-entra-cloud-sync). This is where I come back to the beginning, mentioning Entra Cloud Sync as second option. This approach is a SaaS that works from the cloud and allows to set up and manage their sync preferences online. But before you migrate, please evaluate first, because Cloud Sync does not fully support all hybrid scenarios like Connect Sync: [https://aka.ms/EvaluateSyncOptions](https://aka.ms/EvaluateSyncOptions)

**Stay in sync** 🔄️

…and follow for part two if you want to see how to script your report!