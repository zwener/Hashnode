---
title: "Got lost in PowerShell module versions?"
seoTitle: "Simplifying PowerShell Module Version Management"
seoDescription: "Resolve PowerShell module version conflicts and streamline updates with MicrosoftGraphPS. Achieve harmony in your module ecosystem effortlessly"
datePublished: Mon Sep 29 2025 19:38:51 GMT+0000 (Coordinated Universal Time)
cuid: cmg5j7q9o000002jyge16h0wt
slug: got-lost-in-powershell-module-versions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1759174546015/f75d811d-fa4c-4cbc-b682-c4c37eef2bf0.png
tags: powershell, scripting

---

Hopefully I am not the only guy struggling with dll conflicts created by incompatible PowerShell modules and it\`s different versions. I am not wondering about these circumstances because I assume every technical guy who’s scripting a bit is more creative than interested in house keeping. So over the time I update my PowerShell modules - also with a -Force parameter to be honest. Modules which I have just used once or twice remain on their place and of course I do not clean up my older versions - until now! Time has come to overcome the module chaos - and maybe you can use it for yourself:

But let’s start with the issue. Maybe you guys have encountered the following error:

`Could not load file or assembly 'C:\Users\User\UsersOneDrive\Documents\PowerShell\Modules\ExchangeOnlineManagement\3.9.0\netCore\Microsoft.Identity.Client.dll'. The located assembly's manifest definition does not match the assembly reference.(0x80131040)`

I faced that today (once again) after updating some modules. Thankfully I found an PowerShell module called [**MicrosoftGraphPS**](https://github.com/KnudsenMorten/MicrosoftGraphPS) created by Microsoft MVP [Morten Knudsen](http://aka.ms/morten).

To get your PowerShell house keeping done, just install the module (I just ran it in my user context):

```powershell
Install-Module -Name MicrosoftGraphPS -Repository PSGallery -Force -Scope CurrentUser
```

Then you can run it with the following options:

1. Show details only
    
    ```powershell
    Manage-Version-Microsoft.Graph -Scope CurrentUser
    ```
    
2. Show details, install latest and clean-up old versions
    
    ```powershell
    Manage-Version-Microsoft.Graph -CleanupOldMicrosoftGraphVersions -Scope CurrentUser
    ```
    
3. Install, Update and Clean-up older Microsoft Graph versions (except the latest available version)
    
    ```powershell
    Manage-Version-Microsoft.Graph -InstallLatestMicrosoftGraph -CleanupOldMicrosoftGraphVersions -Scope CurrentUser 
    ```
    

I stick to these options because these are the most practical for me. There are some more enforce a re-installation or remove all versions of the Graph modules (but that’s not happening EVER 😜). Check out the readme of the module for further details. You can also use a scheduled task to implement an automatic update cycle. At the moment I think I stick to the manual process, but let’s see…

After firing these cmdlets everything worked again and all conflicts are gone. In my case 18 older versions were removed:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1759171740541/ed874a62-3f98-4793-bbec-163529d60df5.png align="center")

How do you manage your modules and compatibilities? Please comment your approach. And now enjoy being on the latest and greatest version - also of yourself 👑