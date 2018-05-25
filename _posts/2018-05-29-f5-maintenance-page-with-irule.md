---
title: "F5 iRules"
comments: false
description: "F5 maintenance page with iRule"
keywords: "F5, iRules, example"
published: false
---



#### Create maintenance page with iRule using HTML iFile

1.	Make sure in which partition you want to upload html file. It make things easier if VIP, iRule and iFile are in the same partition.
2.	Go to `System > File Management > iFile List`. Click on Import and upload your html file (maintenance.html) and give it a Name (maintenance.html).
3.	Go to `Local Traffic > iRules > iFile List`. Click on Create, select the iFile you have just uploaded from File Name and give it a Name (maintenance.html)
4.	Go to `Local Traffic > iRules > iRules List`. `Create` a new iRule and give it a Name (maintenance). Here is a sample iRule which will display maintenance html page when all  pool members are down (this decision will be made based on `HTTP Profile` configuration);

```
when LB_FAILED {
    if { [active_members [LB::server pool]] == 0 } {
    HTTP::respond 503 content [ifile get " maintenance.html"]}
    }
```
5.	Go to `Local Traffic > Virtual Servers > Virtual Servers List`. Select the VIP you want to apply iRule to and go to Resource Tab. In the iRule section, click on Manage, add the iRule and click Finished.

NB: If your html file is in a different partition, then you have to use something like `/Common/maintenance.html` in the iRule. We are also using `503` because it will tell search crawler not to cache this page.







#### URL/URI  
URL consists of HOSTNAME and URI ->> `URL = https://[HTTP::host][HTTP::uri]`  
URI consists of PATH and QUERY ->> `[HTTP::uri] = [HTTP::path][HTTP::query]`  

Example: `https://example.com/admin/login.html?service=discovery.com/loginID=8598495`  

where;  
`[HTTP::host]` = `example.com`  
`[HTTP::uri]` = `/admin/login.html?service=discovery.com/loginID=8598495` (everything after hostname)  
`[HTTP::path]` = `/admin/login.html` (everything after hostname and before ?)  
`[HTTP::query]` = `service=discovery.com/loginID=8598495` (everything after ?)  

---

#### Permanent redirect all traffic to a new host but keep the URI  

You have:  
`https://example.com/`  
`https://example.com/apps/login.jsp`  

You want:  
`https://example.com/` to be redirected to `https://discovery.com`  
`https://example.com/apps/login.jsp` to be redirected to `https://discovery.com/apps/login.jsp`  

```
when HTTP_REQUEST {
  HTTP::respond 301 Location "https://discovery.com[HTTP::uri]"
}
```
To do a temporary redirect, use 302 as `HTTP::respond`.

---

#### Redirect only root path(/) to a different host without redirecting nested URI's  

You have:  
`https://example.com/`  
`https://example.com/index.html`  
`https://example.com/apps/login.jsp`  

You want:  
`https://example.com/` to be redirected to `https://discovery.com`  
`https://example.com/index.html` to be redirected to `https://discovery.com`  
`https://example.com/apps/login.jsp` will stay as it is  

```
when HTTP_REQUEST {
  switch -glob [string tolower [HTTP::uri]] {
    "" -
    "/" {
      HTTP::redirect "https://discovery.com"
    }
    "/index.html" {
      HTTP::redirect "https://discovery.com"
    }
    default {
    }
  }
}
```

---

#### URL redirect while keeping HTTP query

You have:  
`https://example.com/admin/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/developer/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/user/login.html?service=discovery.com/loginID=8598495`

You want:  
`https://example.com/admin/login.html?service=discovery.com/loginID=8598495` to be redirected to `https://example.com/admin_new/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/developer/login.html?service=discovery.com/loginID=8598495` to be redirected to `https://example.com/developer_new/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/user/login.html?service=discovery.com/loginID=8598495` to be redirected to `https://example.com/user_new/login.html?service=discovery.com/loginID=8598495`  

```
when HTTP_REQUEST {
  switch -glob [string tolower [HTTP::path]] {
    "" -
    "/" {
      HTTP::redirect https://[HTTP::host]/console/login.html
    }
    "/admin/login.html" {
      HTTP::redirect https://[HTTP::host]/admin_new/login.html?[HTTP::query]
    }
    "/developer/login.html" {
      HTTP::redirect https://[HTTP::host]/developer_new/login.html?[HTTP::query]
    }
    "/user/login.html" {
      HTTP::redirect https://[HTTP::host]/user_new/login.html?[HTTP::query]
    }
    default {
    }
  }
}
```

