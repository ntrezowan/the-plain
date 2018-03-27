
---
title: "irule"
comments: false
description: "irule"
keywords: "irule"
published: false
---
####iRule URL redirect while keeping HTTP query

You have:  
`https://example.com/admin/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/developer/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/user/login.html?service=discovery.com/loginID=8598495`

You want: 
`https://example.com/admin/login.html?service=discovery.com/loginID=8598495` to be redirected to `https://example.com/admin_new/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/developer/login.html?service=discovery.com/loginID=8598495` to be redirected to `https://example.com/developer_new/login.html?service=discovery.com/loginID=8598495`  
`https://example.com/user/login.html?service=discovery.com/loginID=8598495` to be redirected to `https://example.com/user_new/login.html?service=discovery.com/loginID=8598495`  

Concept:
URL consists of HOSTNAME and URI ->> URL: `https://[HTTP::host]/[HTTP::uri]`  
URI consists of PATH and QUERY ->> `[HTTP::uri] = [HTTP::path][HTTP::query]`

So in our example, URL ->>  
`[HTTP::host]` = example.com  
`[HTTP::uri]` = /admin/login.html?service=discovery.com/loginID=8598495 (everything after hostname)  
`[HTTP::path]` = /admin/login.html (everything after hostname and before ?)  
`[HTTP::query]` = service=discovery.com/loginID=8598495 (everything after ?)  

iRule:
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
      HTTP::redirect https://[HTTP::host]/developer_new/logout_dev.html?[HTTP::query]
    }
    "/user/login.html" {
      HTTP::redirect https://[HTTP::host]/user_new/logout_tst.html?[HTTP::query]
    }
    default {
    }
  }
}
```
