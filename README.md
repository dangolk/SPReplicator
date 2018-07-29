# SPReplicator

<img align="left" src=https://user-images.githubusercontent.com/8278033/42554599-39b769a6-8481-11e8-8b6a-379f4a3e54e6.png alt="SPReplicator logo">SPReplicator is a PowerShell module that helps replicate SharePoint list data. 

This module uses the SharePoint Client Side Object Model (CSOM) and all required libraries and dlls are included. Installing the SharePoint binaries is **not required** for the replication to work 👍 Thank you Microsoft for the redistributable nuget.

SPReplicator works with both on-prem and SharePoint Online and is currently in beta. Please report any issues to clemaire@gmail.com.

## Installer
SPReplicator is now in the PowerShell Gallery. Run the following from an administrative prompt to install SPReplicator for all users:
```powershell
Install-Module SPReplicator
```

Or if you don't have administrative access or want to save it locally (just for yourself), run:
```powershell
Install-Module SPReplicator -Scope CurrentUser
```

If you're scheduling tasks via Task Schedule or SQL Server agent, installing the module with administrative privileges is best because it will ensure all users have access via Program Files.

## Usage scenarios

This module can be used for replicating data in a number of ways.

* Between air gapped (offline) servers that do not have direct access to each other
* Directly from SharePoint site collection to SharePoint site collection
* From SQL Server to SharePoint
* From SharePoint to SQL Server
* From CSV to SharePoint
* From SharePoint to CSV

## Usage examples

SPReplicator has a number of commands that help you manage SharePoint lists. You can view, delete, and add records easily and there's even a command that makes it easy to see internal column names and datatypes.

#### Export from SharePoint List

```powershell
Export-SPRListItem -Site https://intranet -List Employees -Path \\nas\replicationdata\Employees.csv
```

#### Establish a session to the SharePoint site

You can specify `-Site` and `-Credential` with every command. Or you can establish a connection and not worry about specifying the Site or Credentials in subsequent command executions.

There is no need to assign the output to a variable, as it creates a reusable global variable `$global:spsite`.

```powershell
# using your own account credentials
Connect-SPRSite -Site https://intranet

# specifying other credentials
Connect-SPRSite -Site https://intranet -Credential ad\otheruser

# using your own account credentials and SP Online
Connect-SPRSite -Site https://corp.sharepoint.com -Credential otheruser@corp.onmicrosoft.com
```

#### Import to SharePoint List
Now that we've established a connection via `Connect-SPRSite`, we no longer need to specify the Site.

We can import data two ways, using `Import-SPRListItem` or `Add-SPRListItem`

```powershell
# Import from CSV
Import-SPRListItem -List Employees -Path \\nas\replicationdata\Employees.csv

# Import from SQL Server
Invoke-DbaSqlQuery -SqlInstance sql2017 -Query "Select fname, lname where id > 100" | Add-SPRListItem -List emps

# Import any PowerShell object, really. So long as it has the properly named columns.
Get-ADUser -Filter * | Select SamAccountName, whateverelse | Add-SPRListItem -List ADList

# Didn't have time to create a good SharePoint list? Use -AutoCreateList
Get-ADUser -Filter * | Add-SPRListItem -List ADList -AutoCreateList

```

The rest of the commands, you can see in screenshots.

## Command summaries

In the screenshots and examples below, I'll be connecting to my SharePoint 2016 server, aptly named `https://sharepoint2016`.

## Connect-SPRSite
Creates a reusable SharePoint Client Context object that lets you use and manage the site collection in Windows PowerShell.

```powershell
Connect-SPRSite -Site https://sharepoint2016
```

![image](https://user-images.githubusercontent.com/8278033/42564673-1ceca0a4-849d-11e8-8f6b-22c1a0aad1e1.png)

## Disconnect-SPRSite
Disconnects a SharePoint Client Context object that lets you use and manage the site collection in Windows PowerShell.

```powershell
Disconnect-SPRSite
```

![image](https://user-images.githubusercontent.com/8278033/42565445-292606ce-849f-11e8-94ee-3986c54441de.png)

## Get-SPRConnectedSite
Returns the connected SharePoint Site Collection.

```powershell
Get-SPRConnectedSite
```

![image](https://user-images.githubusercontent.com/8278033/42572134-b452d4a6-84b4-11e8-9a1e-834c1170a925.png)

## Add-SPRColumn
Adds a column to a SharePoint list.

```powershell
Add-SPRColumn -List 'My List' -ColumnName TestColumn -Description Awesome
```

![image](https://user-images.githubusercontent.com/8278033/42560633-c61f0a78-8492-11e8-9ac2-f3b772d8b8dc.png)

## Add-SPRListItem
Adds items to a SharePoint list from various sources. CSV, generic objects, exported lists.

```powershell
# Add from an object - what's important is that the columns match up, so Title and TestColumn
# exist both in the object and the list.
$object = @()
$object += [pscustomobject]@{ Title = 'Hello'; TestColumn = 'Sample Data'; }
$object += [pscustomobject]@{ Title = 'Hello2'; TestColumn = 'Sample Data2'; }
$object += [pscustomobject]@{ Title = 'Hello3'; TestColumn = 'Sample Data3'; }

$object | Add-SPRListItem -List 'My List'
```

![image](https://user-images.githubusercontent.com/8278033/42570287-227a3c4a-84af-11e8-9e5a-4dc6e9f2f4af.png)

```powershell
# You can even import from a SQL Server database. Note again the Title and TestColumn columns
Invoke-DbaSqlQuery -SqlInstance sql2017 -Query "Select Title = 'Hello SQL', TestColumn = 'Sample SQL Data'"  | 
Add-SPRListItem -List 'My List'
```

![image](https://user-images.githubusercontent.com/8278033/42570441-9a4b0466-84af-11e8-87fc-c04b545e18c9.png)

This is particularly cool. List doesn't exist? Auto-create it! Note, it mostly defaults to text rows so use this sparingly.
```powershell
Invoke-DbaSqlQuery -SqlInstance sql2017 -Query "Select Title = 'Hello SQL',TestColumn = 'Sample SQL Data'"  | 
Add-SPRListItem -List BrandNewList -AutoCreateList
```

![image](https://user-images.githubusercontent.com/8278033/42570505-d060d8be-84af-11e8-948d-f97888611346.png)

![image](https://user-images.githubusercontent.com/8278033/42570634-3f2478fa-84b0-11e8-8ab7-3c996d29021d.png)

## Update-SPRListData
Updates modified items in a SharePoint list.

```powershell
# Update 'My List' from modified rows contained within C:\temp\mylist-updated.xml
$updates = Import-CliXml -Path C:\temp\mylist-updated.xml
Get-SPRListItem -List 'My List' | Update-SPRListItem -UpdateObject $updates -Confirm:$false
```

![image](https://user-images.githubusercontent.com/8278033/42782751-8915d362-88e5-11e8-801e-5987ff7dc89f.png)

## Clear-SPRListItems
Deletes all items from a SharePoint list.

```powershell
# Delete all rows, clearing the list. This will prompt for confirmation.
Clear-SPRListItems -List 'My List'

# Positive you're deleting the rows you want? Add -Confirm:$false to avoid confirmation prompts.
Clear-SPRListItems -List 'My List' -Confirm:$false
```

![image](https://user-images.githubusercontent.com/8278033/42567696-4798dc4c-84a6-11e8-947e-58bff29bbd89.png)

![image](https://user-images.githubusercontent.com/8278033/42567757-7b428f84-84a6-11e8-8863-b654c59044c2.png)

## Export-SPRListItem
Exports all items from a SharePoint list to a file.

```powershell
# Export an entire list
Export-SPRListItem -List 'My List' -Path C:\temp\mylist.xml

# Export only some items
Get-SPRListItem -List 'My List' | Where Title -match Hello2 | Export-SPRListItem -Path C:\temp\hello2.xml
```

The entire list

![image](https://user-images.githubusercontent.com/8278033/42569683-0dda065a-84ad-11e8-8edc-d35058e4e00c.png)

And only some items

![image](https://user-images.githubusercontent.com/8278033/42569711-271efd96-84ad-11e8-8aaa-071c1bbd33a9.png)

## Get-SPRColumnDetail
Returns information (Name, DisplayName, Data type) about columns in a SharePoint list.

```powershell
Get-SPRColumnDetail -List 'My List'
```

![image](https://user-images.githubusercontent.com/8278033/42567935-19fcb8ac-84a7-11e8-9b48-0da67dd2ce0f.png)

## Get-SPRList
Returns a SharePoint list object.

```powershell
Get-SPRList -List 'My List'
```

![image](https://user-images.githubusercontent.com/8278033/42568030-65cde896-84a7-11e8-8a7f-a730f4f26344.png)

## Get-SPRListItem
Returns data from a SharePoint list.

```powershell
# Get the entire list
Get-SPRListItem -List 'My List'

# Get only item 1. You could also get -Id 1, 2, 3 and so on.
Get-SPRListItem -List 'My List' -Id 1
```

![image](https://user-images.githubusercontent.com/8278033/42566521-91a9a7d4-84a2-11e8-9a96-f6765ad3a8aa.png)

![image](https://user-images.githubusercontent.com/8278033/42566593-c7494f70-84a2-11e8-8c1f-17c2054b4b8f.png)

## Get-SPRListTemplate
Get list of SharePoint templates.

```powershell
Get-SPRListTemplate
```

![image](https://user-images.githubusercontent.com/8278033/42564578-d33b9870-849c-11e8-9977-73d061f5d58c.png)

## Import-SPRListItem
Imports all items from a file into a SharePoint list.

```powershell
# Manually specify the path to the xml file, which was exported from Export-SPRListItem
Import-SPRListItem -List 'My List' -Path C:\temp\mylist.xml

# Or pipe it in
Get-ChildItem C:\temp\mylist.xml | Import-SPRListItem -List 'My List' 

# You can even automatically create a list if it doesn't exist
dir 'C:\temp\My List.xml' | Import-SPRListItem -List Test -AutoCreateList
```

![image](https://user-images.githubusercontent.com/8278033/42569956-fb412f40-84ad-11e8-9c25-d7b06470301e.png)

![image](https://user-images.githubusercontent.com/8278033/42579927-092a00ca-84c5-11e8-81e4-2ac501227c71.png)


## New-SPRList
Creates a new SharePoint list.

```powershell
# Create a generic list with a description
New-SPRList -List List1 -Description "My awesome list"

# Create a document library
New-SPRList -List 'My Documents' -Template DocumentLibrary
```
![image](https://user-images.githubusercontent.com/8278033/42560182-c8fd276c-8491-11e8-8c2e-2234b249439c.png)

![image](https://user-images.githubusercontent.com/8278033/42560506-826f8cee-8492-11e8-89a5-0b1eaac26a95.png)

## Remove-SPRList
 Deletes lists from a SharePoint site collection.
 
```powershell
# Delete the list and prompt for confirmation.
Remove-SPRList -List List1

# Positive you're deleting the list you want? Add -Confirm:$false to avoid confirmation prompts.
Remove-SPRList -List List2 -Confirm:$false
```
![image](https://user-images.githubusercontent.com/8278033/42563954-32927cfa-849b-11e8-9ab1-3b973ff098e7.png)

## Remove-SPRListItem
Deletes items from a SharePoint list.

```powershell
# Delete a couple items and prompt for confirmation.
Get-SPRListItem -List 'My List' -Id 44, 45 | Remove-SPRListItem

# Delete a bunch of items without confirmation.
Get-SPRListItem -List 'My List' | Where Title -match Hello | Remove-SPRListItem -Confirm:$false
```

![image](https://user-images.githubusercontent.com/8278033/42569305-c7273e68-84ab-11e8-85eb-2d34610e5220.png)

![image](https://user-images.githubusercontent.com/8278033/42569374-0952af20-84ac-11e8-88c7-eaf7c0664a82.png)

## Select-SPRObject
Makes it easier to alias columns to select and rename for export.
 
```powershell
# Get two columns, FullName and Created from 'My List'. In the example below, FullName is an alias of Title.
# This makes it easy to import items.xml to a SharePoint with with a FullName column.

Get-SPRListItem -Site intranet.ad.local -List 'My List' | Select-SPRObject -Property 'Title as FullName', Created | Export-SPRObject -Path C:\temp\items.xml
```

![image](https://user-images.githubusercontent.com/8278033/43017926-91910924-8bf3-11e8-8133-5084b4a685a3.png)

## TODO

* <del>Create -Since (via caml)</del>
* <del>-AsUser (force modify create/modified column)</del>
* <del>Move the batches</del>
* <del>Add list view test</del>
* Update readme to have only selected screenshots and then the video
* Demo video
* Set column (based off of update, i imagine? then use set in the command?)
* Explore ListItemCollectionPosition
* Add logging to export to SP or SQL. Actually, just do SP. Keep things that'd make PBI look cool.
* Explore extracting files from attachments
* Explore hooks, such as the check-in or check-out of a document.
* File copy using BITS and zips and maybe Microsoft Remote Differential Compression?

## MAYBE WONT DO

* Make a dictionary that tracks what's already been imported
    * Taken care of by Column Key, but maybe.

## Pester tested

This module comes with integration tests! If you'd like to see how I test the commands, check out [Integration.Tests.ps1](https://github.com/potatoqualitee/SPReplicator/blob/master/tests/Integration.Tests.ps1)

![image](https://user-images.githubusercontent.com/8278033/42579528-41a428e6-84c4-11e8-9bc6-4a987a9679f4.png)

## Learn more

To find out more about any command, including additional examples, use `Get-Help`. 

```powershell
Get-Help Get-SPRColumnDetail -Detailed
```

<!--
$global:a = @()
foreach ($prop in $props.Name) {
    try {
        $server.Load($server.Web.$prop)
    }
    catch {
        $global:a += $prop
        continue
    }
}
-->