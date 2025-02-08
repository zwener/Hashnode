---
title: "Remove invalid retention policy from SharePoint Online"
seoTitle: "Remove invalid retention policy from SharePoint Online"
datePublished: Sat Feb 08 2025 13:13:54 GMT+0000 (Coordinated Universal Time)
cuid: cm6w7v6vt000t09l7bx4v6fnl
slug: remove-invalid-retention-policy-from-sharepoint-online
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1739020887573/d4be876f-7834-49de-9e43-856c93fbd6a9.png
tags: powershell, compliance, sharepoint-online, microsoft365, retention, pnp, retentionpolicy, hold

---

Despite the common sense of using retention features in Microsoft 365 I never had trouble removing data if I had to. But last week I faced something new about invalid retention policies - but lets start from the beginning.

As we all know, sometimes things do not work as expected…in this case somebody uploaded a bunch of files resulting in extensive SharePoint storage usage. And of course the data has been deleted, but the site is configured for retention.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738873265907/1032f669-8c71-448f-a072-31c84cb9a4fc.png align="center")

My general approach of deleting data, which is on hold is more or less the same:

1. If there are Retention Policies in place, exclude the site or group from the retention policy.
    
2. Close existing eDiscovery cases with a hold
    
3. Disable Data loss prevention (DLP) policies, if there are some
    

In my specific case I double-checked all parts of the game, but only had to set the retention policy exclusion. Five minutes later (after the policy status showed success) I browsed the Preservation Hold Library. Important to mention that this library keeps copies of all modified or deleted files as enforced by the Retention policy or eDiscovery holds. You can reach it by using “PreservationHoldLibrary“ as a suffix on your SharePoint site URL: [https://TenantDomain.sharepoint.com/sites/SiteName/PreservationHoldLibrary](https://YourDomain.sharepoint.com/sites/YourSite/PreservationHoldLibrary)

I tried to delete some files…and tried and tried. Every M365 admin knows that 5 minutes is not the appropriate period of time to get things changed. So I kept trying after 24 hours… But the error still told me very kind:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738916791673/73964ae2-cb8f-4ad0-9d63-195dcaf50d62.png align="center")

*“Request was cancelled by event received. If attempting to delete a non-empty folder, it’s possible that it’s on hold.”*

Of course the status of the retention policy was showing success and not pending. So I assumed the group site is excluded. I also tried to get insights with PowerShell and used the [Invoke-HoldRemovalAction](https://learn.microsoft.com/en-us/powershell/module/exchange/invoke-holdremovalaction?view=exchange-ps) cmdlet to find applied holds. But there were no holds applied, so I still had to search for them:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738930922225/a0d23b29-74b4-4ea5-89df-76e97aa8fe00.png align="center")

The version history was still showing that the files are not expiring because they are on hold:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738931125709/5af188b5-b9b5-43e9-b2ae-56b40907dae7.png align="center")

During a helpful Microsoft support case a guy told me to use the “Help & support“ in the M365 Admin Center to find and remove invalid retention policies by searching for: **Remove invalid retention policy from SharePoint or OneDrive**

Just run the test and magic happens: it shows the invalid policy, which still applies:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1738872886765/98eeae09-190c-4d7d-8652-e3beef531860.png align="center")

Just acknowledge and update in the wizard and there you go 🪄 Now I was ready to delete files and so I [let it be, let it be, let it be, let it be…](https://www.youtube.com/watch?v=QDYfEBY9NM4) 🎶

Wow! Lesson learned - this thing is actually good for something!

And just in case you have to delete a lot of files use PowerShell with the [PNP module](https://pnp.github.io/powershell/).

```powershell
$siteURL = "https://yourtenant.sharepoint.com/sites/yoursite"
$listName = "Preservation Hold Library"

# install PNP module [https://pnp.github.io/powershell/articles/installation.html]
Install-Module PnP.PowerShell -Scope CurrentUser

# register EntraID app for PnP PowerShell interactive login approach [https://pnp.github.io/powershell/articles/registerapplication.html]
Register-PnPEntraIDAppForInteractiveLogin -ApplicationName "PnP.App" -Tenant yourtenant.onmicrosoft.com -Interactive

# connect to SPO with PNP
Connect-PnPOnline -Url $siteURL -Interactive -ClientId <client id of your Entra ID Application Registration>

# get list
$list = Get-PnPList -Identity $listName 
 
# retrieves all list items from the list in pages of 1000 items and delete each item
Get-PnPListItem -List $list -PageSize 1000 -ScriptBlock {
        Param($items)
        Invoke-PnPQuery 
        } | ForEach-Object {
            # remove files without confirmation
            Remove-PnPListItem -Identity $_ -Force
        }
```

Time for a shout-out for my favorite site about SharePoint is [https://www.sharepointdiary.com](https://www.sharepointdiary.com/). Finally don’t forget to remove the exclude setting in the retention policy afterwards 😉