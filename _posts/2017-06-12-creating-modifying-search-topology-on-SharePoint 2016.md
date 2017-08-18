---
title: "Creating/Modifying Search Topology on SharePoint 2016"
comments: false
description: "Creating/Modifying Search Topology on SharePoint 2016"
keywords: "sharepoint, 2016, search, topology, create, modify, multiple, search, servers"
---

#### A. Configuring a new Search Topology

__Farm__

| Hostname | Role        | Components                                                              |
|----------|-------------|-------------------------------------------------------------------------|
| App1     | Application | Admin, Crawl, Content Processing, Indexing, Query Processing, Analytics |
| App2     | Application | Admin, Crawl, Content Processing, Indexing, Query Processing, Analytics |
| Web1     | Web Front   | Indexing, Query Processing                                              |
| Web2     | Web Front   | Indexing, Query Processing                                              |


1. Create a new Search application `Search Service Application` with database name `SharePoint_Search`; 
```
$ssa = New-SPEnterpriseSearchServiceApplication -Name "Search Service Application" -DatabaseName "SharePoint_Search" -ApplicationPool "Search Service AppPool" -AdminApplicationPool "Search Service AdminAppPool"

```
2. Create a new proxy for Search application `Search Service Application Proxy`; 
```
New-SPEnterpriseSearchServiceApplicationProxy -Name "Search Service Application Proxy" -SearchApplication $ssa
```

3. Save the Search Service Application into a variable;
```
$ssa = Get-SPEnterpriseSearchServiceApplication
```

4. Define variables (hostA, hostB...) for each servers that will be included in the Search Topology;
```
$hostA = Get-SPEnterpriseSearchServiceInstance -Identity "App1"
$hostB = Get-SPEnterpriseSearchServiceInstance -Identity "App2"
$hostC = Get-SPEnterpriseSearchServiceInstance -Identity "Web1"
$hostD = Get-SPEnterpriseSearchServiceInstance -Identity "Web2"
```

5. Start Search Service in all the servers that will be included in the Search Topology;
```
Start-SPEnterpriseSearchServiceInstance -Identity $hostA
Start-SPEnterpriseSearchServiceInstance -Identity $hostB
Start-SPEnterpriseSearchServiceInstance -Identity $hostC
Start-SPEnterpriseSearchServiceInstance -Identity $hostD
```

6. Verify that Search instance is running in all the servers;
```
Get-SPEnterpriseSearchServiceInstance -Identity $hostA
Get-SPEnterpriseSearchServiceInstance -Identity $hostB
Get-SPEnterpriseSearchServiceInstance -Identity $hostC
Get-SPEnterpriseSearchServiceInstance -Identity $hostD
```

7. Run the following command to create a new topology;
```
$newTopology = New-SPEnterpriseSearchTopology -SearchApplication $ssa
```

8. Configure `App1` and `App2` to have Admin, Crawl, Content Processing, Indexing, Query Processing and Analytics;
```
New-SPEnterpriseSearchAdminComponent -SearchTopology $newTopology -SearchServiceInstance $hostA
New-SPEnterpriseSearchCrawlComponent -SearchTopology $newTopology -SearchServiceInstance $hostA
New-SPEnterpriseSearchContentProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostA
New-SPEnterpriseSearchAnalyticsProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostA
New-SPEnterpriseSearchQueryProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostA
New-SPEnterpriseSearchIndexComponent -SearchTopology $newTopology -SearchServiceInstance $hostA -IndexPartition 0
```
```
New-SPEnterpriseSearchAdminComponent -SearchTopology $newTopology -SearchServiceInstance $hostB
New-SPEnterpriseSearchCrawlComponent -SearchTopology $newTopology -SearchServiceInstance $hostB
New-SPEnterpriseSearchContentProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostB
New-SPEnterpriseSearchAnalyticsProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostB
New-SPEnterpriseSearchQueryProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostB
New-SPEnterpriseSearchIndexComponent -SearchTopology $newTopology -SearchServiceInstance $hostB -IndexPartition 0
```

9. Configure `Web1` and `Web2` to have Query Processing and Indexing;
```
New-SPEnterpriseSearchQueryProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostC
New-SPEnterpriseSearchIndexComponent -SearchTopology $newTopology -SearchServiceInstance $hostC -IndexPartition 0
```
```
New-SPEnterpriseSearchQueryProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostD
New-SPEnterpriseSearchIndexComponent -SearchTopology $newTopology -SearchServiceInstance $hostD -IndexPartition 0
```

10. Activate the topology;
```
Set-SPEnterpriseSearchTopology -Identity $newTopology
```

11. Verify the topology;
```
Get-SPEnterpriseSearchTopology -SearchApplication $ssa
```

12. Check the status of the index components;
```
Get-SPEnterpriseSearchStatus -SearchApplication $ssa -Text
```
___

#### B. Modifying an existing Search Topology

1. Save active search topology into a variable;
```
$ssa = Get-SPEnterpriseSearchServiceApplication
$active = Get-SPEnterpriseSearchTopology -SearchApplication $ssa -Active
```

3. Clone the topology;
```
$clone = New-SPEnterpriseSearchTopology -SearchApplication $ssa -Clone -SearchTopology $active
```

4. Wait for the indexes to be replicated and then we can remove old index component. To remove old index, we need to find `ComponentId`. Run the following command to find `ComponentId`;
```
Get-SPEnterpriseSearchComponent -SearchTopology $clone
```
It will be something like `c8309775-dcf3-4a07-8598-05dbc98b04b0`.

5. Run the following command to remove the component that we have already cloned;
```
Remove-SPEnterpriseSearchComponent -Identity 'c8309775-dcf3-4a07-8598-05dbc98b04b0' -SearchTopology $ clone
```

6. Activate the new cloned topology;
```
$clone.Activate()
```

7. Remove old inactive topology;
```
Get-SPEnterpriseSearchServiceApplication | Get-SPEnterpriseSearchTopology |? {$_.State -eq "Inactive"} |% { Remove-SPEnterpriseSearchTopology -Identity $_ -Confirm:$false}
```

___

**Changing Index path**
Default index location is at `C:\Program Files\Microsoft Office Servers\16.0\Data\Office Server\Applications` which may create issue since it's reading/writing to a slow I/O hard disk. To change the location and move it to a flash drive, run the following command;
```
New-SPEnterpriseSearchIndexComponent -SearchTopology $clone -SearchServiceInstance $ssa -IndexPartition 0 -RootDirectory Z:\FlashDrive\Index\0
```
We need to run this command before activing the topology.




