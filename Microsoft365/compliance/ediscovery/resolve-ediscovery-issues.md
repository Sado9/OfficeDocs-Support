---
title: Troubleshooting common eDiscovery issues
description: Learn about basic troubleshooting steps you can take to resolve common issues in Microsoft Purview eDiscovery.
f1.keywords: 
  - NOCSH
ms.author: v-six
author: simonxjx
manager: dcscontentpm
audience: Admin
ms.topic: troubleshooting
localization_priority: Normal
ms.custom: 
  - CI 157398
  - CI 171536
  - CSSTroubleshoot
  - seo-marvel-apr2020
ms.collection: 
  - Strat_O365_IP
  - M365-security-compliance
search.appverid: 
  - MOE150
  - MET150
siblings_only: true
ms.date: 4/21/2023
---
# Resolve common eDiscovery issues

This article covers basic troubleshooting steps that you can take to identify and resolve issues that you might encounter during an eDiscovery search or elsewhere in the eDiscovery process. For information about how to troubleshoot common retention policy errors, see [Resolve errors in Microsoft 365 retention and retention label policies](/microsoft-365/troubleshoot/retention/resolve-errors-in-retention-and-retention-label-policies).

Resolving some of these scenarios requires help from Microsoft Support. Information about when to contact Microsoft Support is included in the resolution steps.

## Error: Ambiguous location

If you try to add a user's mailbox location to a search, and there are duplicate or conflicting objects that have the same userID in the Exchange Online Protection (EOP) directory, you receive the following error message:

> The compliance search contains the following invalid location(s): `useralias@contoso.com`. The location `useralias@contoso.com` is ambiguous.

### Resolution

Check for duplicate users or distribution lists that have the same user ID:

1. Connect to [Security & Compliance Center PowerShell](/powershell/exchange/connect-to-scc-powershell).

2. To retrieve all instances of the username, run the following command:

    ```powershell
    Get-Recipient <username>
    ```

   The output for "useralias@contoso.com" will resemble the following:

   > |Name|RecipientType|
   > |---|---|
   > |Alias, User|MailUser|
   > |Alias, User|User|

3. If multiple users are returned, locate and fix the conflicting object.

## Error: Search fails on specific locations

An eDiscovery or Content search might yield the following error:

> This search completed with (#) errors.  Would you like to retry the search on the failed locations?

:::image type="content" source="media/resolve-ediscovery-issues/search-fails-error.png" alt-text="Screenshot of search-specific location fails error.":::

### Resolution

If you receive this message, we recommend that you verify the locations that failed in the search, and then rerun the search on the failed locations only:

1. Connect to [Security & Compliance Center PowerShell](/powershell/exchange/connect-to-scc-powershell), and then run the following command:

   ```powershell
   Get-ComplianceSearch <searchname> | FL
   ```

2. From the PowerShell output, view the failed locations in the errors field or from the status details in the error message from the search output.

3. Retry the eDiscovery search on the failed locations only.

4. If you continue to receive these errors, see [Retry failed locations](/Office365/SecurityCompliance/retry-failed-content-search) for more troubleshooting steps.

## Error: File not found

When you run an eDiscovery search that includes SharePoint Online and OneDrive for Business locations, you might receive a "File Not Found" error message even though the file is located on the site. This message will be posted in the export warnings and errors.csv file or skipped items.csv file. This might occur if the file can't be found on the site or if the index is out of date. Here's the text of an actual error message (with emphasis added):

> 28.06.2019 10:02:19_FailedToExportItem_Failed to download content. Additional diagnostic info : Microsoft.Office.Compliance.EDiscovery.ExportWorker.Exceptions.ContentDownloadTemporaryFailure: Failed to download from content 6ea52149-xyxy-xyxy-b5bb-82ca6a3ec9be of type Document. Correlation Id: 3bd84722-xyxy-xyxy-b61b-08d6fba9ec32. ServerErrorCode: -2147024894 ---> Microsoft.SharePoint.Client.ServerException: ***File Not Found***. at Microsoft.SharePoint.Client.ClientRequest.ProcessResponseStream(Stream responseStream) at Microsoft.SharePoint.Client.ClientRequest.ProcessResponse()
--- End of inner exception stack trace ---

### Resolution

1. Check the location that's identified in the search to make sure that the location of the file is correct and added to the search locations.

2. To reindex the site, use the procedures that are provided in [Manually request crawling and reindexing of a site, a library, or a list](/sharepoint/crawl-site-content).

## Error: This file wasn't exported 

**Full error message:** "This file wasn't exported because it doesn't exist anymore. The file was included in the count of estimated search results because it's still listed in the index. The file will eventually be removed from the index, and won't cause an error in the future."

You might experience this error when you run an eDiscovery search that includes SharePoint Online and OneDrive for Business locations. eDiscovery relies on the SPO index to identify the file locations. If the file was deleted but the SPO index was not yet updated, this error might occur.

### Resolution 

Open the SPO location, and verify that this file is, indeed, not there. The suggested solution is to manually reindex the site, or wait until the site is reindexed through the automatic background process.

## Error: This search result was not downloaded

**Full error message:** "This search result was not downloaded as it is a folder or other artifact that can't be downloaded by itself, any items inside the folder or library will be downloaded."

You might experience this error when you run an eDiscovery search that includes SharePoint Online and OneDrive for Business locations. This message means that we were going to try to export the item that's reported in the index, but the item turned out to be a folder. Therefore, we did not export it. As mentioned in the error message, we don't export folders, but we do export their contents.

## Error: Search fails because recipient is not found

An eDiscovery search fails and returns a "recipient not found" error message. This error might occur if the user object cannot be found in Exchange Online Protection (EOP) because the object is not synced.

### Resolution

1. Connect to [Exchange Online PowerShell](/powershell/exchange/connect-to-exchange-online-powershell).

2. Run the following command to check whether the user is synced to Exchange Online Protection:

   ```powershell
   Get-Recipient <userId> | FL
   ```

3. There should be a mail user object for the user question. If nothing is returned, investigate the user object. If the object can't be synced, contact Microsoft Support.

## Error: Search fails with error CS007

When you run a Content search or a search that's associated with an eDiscovery (Standard) case, a transient error occurs, and the search fails and returns a "CS007" error message.

### Resolution

1. Update the search, and reduce the complexity of the search query. For example, a wildcard search might return too many results for the system to process, and this triggers a CS007 error.

2. Rerun the updated search.

## Error: Exporting search results is slow

When you export search results from an eDiscovery (Standard) or Content search in the Microsoft Purview compliance portal, the download takes longer than expected. You can check to see the amount of data to be download, and consider increasing the export speed.

### Resolution

1. Connect to [Security & Compliance Center PowerShell](/powershell/exchange/connect-to-scc-powershell), and then run the following command:

   ```powershell
   Get-ComplianceSearch <searchname> | FL
   ```

2. To determine the amount of data that has to be downloaded, examine the `SearchResults` and `SearchStatistics` parameters.

3. Run the following command:

   ```powershell
   Get-ComplianceSearchAction | FL
   ```

4. In the results field, find the data that was exported, and then view any messages about errors that were encountered.

5. Open the folder that you exported the content to, and examine the Trace.log file for any error entries.

6. If you still experience issues, consider subdividing any searches that return a large set of results. For example, you can use date ranges in search queries to create a set of smaller searches that return fewer results and can be downloaded faster.

## Error: Export process not progressing or is stuck

When you export search results from eDiscovery (Standard) or Content search in the Microsoft Purview compliance portal, the export process does not progress, or it appears to be stuck.

### Resolution

1. If it's necessary, rerun the search. If the search was last run more than seven days ago, you have to rerun the search.

2. Restart the export.

## Error: "Request Failed with status Code 500" or "500 internal server error"

When you perform an eDiscovery export, you receive one of the following error messages:

- Request Failed with status Code 500
- 500 internal server error

### Resolution

1. Make sure that you're assigned the **Export** role. This role is assigned automatically to the **eDiscovery Manager** role group. If you're a member of the **Organization Management** role group, add yourself to the **eDiscovery Manager** role group on the **Permissions** page in the Microsoft Purview compliance portal. You can also view your permissions on the eDiscovery (Premium) overview page in the compliance portal. For more information about eDiscovery permissions, see [Assign eDiscovery permissions in the Microsoft Purview compliance portal](/microsoft-365/compliance/ediscovery-assign-permissions?view=o365-worldwide&preserve-view=true).
1. If you receive an error message when you try to download the export, verify that you're downloading the export that you created. You might have to contact the administrator who created the export to complete the download.
1. Check security filters and the storage location.
1. Subdivide the search into smaller searches by using a smaller date range or by limiting the number of locations that are searched. Then, run the search again.

   **Note:** In Content Search and eDiscovery (Standard), the maximum exportable data for a single search is 2 TB. If your export exceeds 2 TB, reduce the amount of data to be exported, and then try again.

## Error: Holds don't sync

This is an eDiscovery Case Hold Policy Sync Distribution error. When this occurs, you receive the following error message:

> Resources: It's taking longer than expected to deploy the policy. It might take an additional 2 hours to update the final deployment status, so check back in a couple hours.

### Resolution

1. Connect to [Security & Compliance Center PowerShell](/powershell/exchange/connect-to-scc-powershell), and then run the following command for an eDiscovery case hold:

   ```powershell
   Get-CaseHoldPolicy <policyname> - DistributionDetail | FL
   ```

2. Examine the value in the `DistributionDetail` parameter for an error entry such as the following:

   > Error: Resources: It's taking longer than expected to deploy the policy. It might take an additional 2 hours to update the final deployment status, so check back in a couple hours."

3. Try running the RetryDistribution parameter on the policy in question:

   For eDiscovery case holds:

   ```powershell
   Set-CaseHoldPolicy <policyname> -RetryDistribution
   ```

4. Contact Microsoft Support.

## Error: Holds stuck in PendingDeletion

In some situations, eDiscovery Case Hold Policies are stuck in PendingDeletion and can't be removed.

### Resolution

1. Connect to [Security & Compliance Center PowerShell](/powershell/exchange/connect-to-scc-powershell).
1. Try running the RetryDistribution parameter on the policy in question:

   For eDiscovery case holds:

   ```powershell
   Set-CaseHoldPolicy <policyname> -RetryDistribution
   ```

2. Try to delete the policy by using PowerShell and the `-ForceDeletion` parameter:

   For eDiscovery case holds, use the [Remove-CaseHoldPolicy](/powershell/module/exchange/remove-caseholdpolicy?view=exchange-ps&preserve-view=true) cmdlet:

   ```powershell
   Remove-CaseHoldPolicy <policyname> -ForceDeletion
   ```
  
3. Contact Microsoft Support.

## Error: "The condition specified using HTTP conditional header(s) is not met"

When you try to download search results by using the eDiscovery Export Tool, you receive the following error message:

> System.Net.WebException: The remote server returned an error: (412) The condition specified using HTTP conditional header(s) is not met.

This is a transient error that typically occurs in the Azure Storage location.

### Resolution

To resolve this issue, try again [download the search results](/microsoft-365/compliance/export-search-results#step-2-download-the-search-results). This will restart the eDiscovery Export Tool.

## Error: Downloaded export shows no results

After a successful export, the download that was made through the eDiscovery Export Tool shows zero files in the results.

### Resolution

This is a client-side issue. To fix the issue, follow these steps:

1. Try using another client to download.

2. Remove old searches that are no longer required. To do this, run the [Remove-ComplianceSearch](/powershell/module/exchange/remove-compliancesearch) cmdlet.

3. Make sure to download to a local drive.

4. Make sure that no virus scanner is running.

5. Make sure that no other export is downloading to the same folder or any parent folder.

6. If the previous steps don't work, disable file compression and de-duplication.

7. If step 6 works, then the issue occurs because of a local virus scanner or a disk issue.

## Error: "Your request can't be started because the maximum number of jobs for your organization are currently running"

Your organization has reached the limit for the maximum number of concurrent export jobs. All new export jobs are being throttled.

### Resolution

To discover how many export jobs that were started in the past seven days are still running, follow these steps:

1. Connect to [Security & Compliance Center PowerShell](/powershell/exchange/connect-to-scc-powershell).

2. To collect information about the current export jobs that are triggering the throttle, run the following cmdlets as an eDiscovery administrator.

    **Note**: An eDiscovery administrator is a member of the eDiscovery Manager role group, and can view all eDiscovery cases. You can use the [Get-eDiscoveryCaseAdmin](/powershell/module/exchange/get-ediscoverycaseadmin) cmdlet to check for eDiscovery administrators, and use the [Add-eDiscoveryCaseAdmin](/powershell/module/exchange/add-ediscoverycaseadmin) cmdlet to add an eDiscovery administrator. The cmdlets might take some time to finish, depending on the number of cases.

   ```powershell
   $date = Get-Date
   $Exports = @(Get-ComplianceSearchAction -export -ResultSize Unlimited)
   $cases = Get-ComplianceCase | ?{$_.status -like "Active"}
     
   $i = 1
   foreach ($case in $cases)
   {
   $Exports += Get-ComplianceSearchAction -export -case $case.name
   write-host "Processing case $($i) of $($cases.count)"
   $i++
   }
     
   $inprogressExports = $exports | ?{$_.Results -eq $null -or (!$_.Results.Contains("Export status: Completed") -and !$_.Results.Contains("Export status: none"))};
   $exportJobsRunning = $inprogressExports | ?{$_.JobStartTime -ge $date.AddDays(-7)} | Sort-Object JobStartTime -Descending

   ```

3. Run the following cmdlet to display a list of export jobs that are currently running.

   **Note**: If the cmdlet returns 10 or more exports jobs, your organization has reached the limit for the number of concurrent export jobs. For more information, see [Limits for eDiscovery search](/microsoft-365/compliance/limits-for-content-search).

   ```powershell
   $exportJobsRunning | Format-Table Name, JobStartTime, JobEndTime, Status | More;
   ```

4. Wait for existing export jobs to finish, or use the [Remove-ComplianceSearchAction](/powershell/module/exchange/remove-compliancesearchaction) cmdlet to remove export jobs that are no longer required.

## Error: Hit tolerable error, will retry

**Full error message:** "Hit tolerable error, will retry: The process cannot access the file 'ExportData.db' because it is being used by another process."

The export process might get stuck or produce zero-byte files.

### Resolution

This might be a client-side issue. To fix it, follow these steps:

1. Try using another client to download.

2. Remove old searches that are no longer required. To do this, run the [Remove-ComplianceSearch](/powershell/module/exchange/remove-compliancesearch) cmdlet.

3. Make sure to download to a local drive.

4. Make sure that no virus scanner is running.

5. Make sure that no other export is downloading to the same folder or any parent folder.

6. If the previous steps don't work, disable file compression and de-duplication.

7. If step 6 works, then the issue occurs because of a local virus scanner or a disk issue.

If none of these steps solve the problem, gather the output of the `Get-ComplianceSearch` and `Get-ComplianceSearchAction` parameters before you create a support case.

## Error: Problem retrieving mailbox items while archiving

The following error entries are displayed in the Export Warnings.csv and Errors.csv files during a Content search and the eDiscovery standard export workflow.

> "FailedToExportItem_Microsoft.Exchange.EDiscovery.Export.ExportException: Export failed with error type: 'FailedToExportItem'. Message: Item has been moved or deleted."
> "FailedToExportItem_Microsoft.Exchange.EDiscovery.Export.ExportException: Export failed with error type: 'FailedToExportItem'. Message: Unable to retrieve item due to timeout after multiple retries."

These errors entries indicate that certain items that were found during a search couldn’t be retrieved. These might be temporary backup copies that are created during archiving. Although these temporary backups are accessible for searching and, therefore, can be matched, they are not accessible for retrieval. However, eDiscovery can match and retrieve the original items because they are exact copies of the backups.

### Resolution

No action is needed to address these errors. The original items that are associated with the same mailbox will be retrieved and subsequently exported or added to a review set.

## Error: Export process opens a new blank page without a download

When you export search results from an eDiscovery (Standard) or Content search in the Microsoft Purview compliance portal, the export process opens a new blank page, and the eDiscovery Export Tool (UnifiedExportTool) isn't downloaded.

### Resolution

To fix this issue, allow pop-ups from the Microsoft Purview compliance portal.

In Microsoft Edge:

1. Select the **Settings and more** icon (...) in the upper-right corner of the browser.
1. Select **Settings** > **Cookies and site permissions**.
1. Under **All permissions**, select **Pop-ups and redirects**.
1. Under **Allow**, select **Add**.
1. In the **Add a site** dialog box, enter *https://compliance.microsoft.com*, and then select **Add**.

If you're using a different browser, follow the browser's documentation to allow pop-ups from the Microsoft Purview compliance portal.

After you make this change, restart the export process.
