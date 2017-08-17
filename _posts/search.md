1. Save Search Service Application into a variable;
```
$ssa = Get-SPEnterpriseSearchServiceApplication
```

2. Define variables (hostA, hostB...) for each servers that will be included in the Search Topology;
```
$hostA = Get-SPEnterpriseSearchServiceInstance -Identity "App1"
$hostB = Get-SPEnterpriseSearchServiceInstance -Identity "App2"
$hostC = Get-SPEnterpriseSearchServiceInstance -Identity "web1"
$hostD = Get-SPEnterpriseSearchServiceInstance -Identity "web2"
```

3. Start Search Service in all the servers that will be included in the Search Topology;
```
Start-SPEnterpriseSearchServiceInstance -Identity $hostA
Start-SPEnterpriseSearchServiceInstance -Identity $hostB
Start-SPEnterpriseSearchServiceInstance -Identity $hostC
Start-SPEnterpriseSearchServiceInstance -Identity $hostD
```

4. Verify if Search instance is running in all the servers;
```
Get-SPEnterpriseSearchServiceInstance -Identity $hostA
Get-SPEnterpriseSearchServiceInstance -Identity $hostB
Get-SPEnterpriseSearchServiceInstance -Identity $hostC
Get-SPEnterpriseSearchServiceInstance -Identity $hostD
```

5. If it is a new Search Service Application and no topology exist, then run the following command to create a new topology;
```
$newTopology = New-SPEnterpriseSearchTopology -SearchApplication $ssa
```

6. If we have an exisitng topology, then clone it. Search service can have multiple topology but only one can be active a time. We will activate our current topology later;
```
$clone = $ssa.ActiveTopology.Clone()
```

7. Configure `App1` and `App2` to have Admin, Crawl, Content Processing, Indexing, Query Processing and Analytics;
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

8. Configure `Web1` and `Web2` to have Query Processing and Indexing;
```
New-SPEnterpriseSearchQueryProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostC
New-SPEnterpriseSearchIndexComponent -SearchTopology $newTopology -SearchServiceInstance $hostC -IndexPartition 0
```
```
New-SPEnterpriseSearchQueryProcessingComponent -SearchTopology $newTopology -SearchServiceInstance $hostD
New-SPEnterpriseSearchIndexComponent -SearchTopology $newTopology -SearchServiceInstance $hostD -IndexPartition 0
```

9. Activate the new topology;
```
Set-SPEnterpriseSearchTopology -Identity $newTopology
```

10. Verify the topology;
```
Get-SPEnterpriseSearchTopology -SearchApplication $ssa
```


