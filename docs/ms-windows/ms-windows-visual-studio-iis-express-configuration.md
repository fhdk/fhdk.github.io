---
published: true
date: '2020-08-01 00:00'
publish_date: '2020-08-01 00:00'
title: 'Visual Studio IISExpress'
taxonomy:
    category:
        - docs
metadata:
    author: linux-aarhus
---

## LAN access

Elevated prompt

```
netsh http add urlacl url=http://ipadress:port/ user=everyone
netsh http add urlacl url=http://hostname:port/ user=everyone
netsh http add urlacl url=http://hostname.domain.tld:port/ user=everyone
```    

Edit iis **applicationhost.config** for project 

> **sln\\.vs\\sln\\config\\applicationhost.config**
 
 ```
<site name="siteName" id="6">
  <application path="/" applicationPool="Clr4IntegratedAppPool">
    <virtualDirectory path="/" physicalPath="physicalPath" />
  </application>
  <bindings>
    <binding protocol="http" bindingInformation="*:port:localhost" />
    <binding protocol="https" bindingInformation="*:port:localhost" />
    <binding protocol="http" bindingInformation="*:port:hostname" />
    <binding protocol="https" bindingInformation="*:port:hostname" />
    <binding protocol="http" bindingInformation="*:port:hostname.domain.tld" />
    <binding protocol="https" bindingInformation="*:port:hostname.domain.tld" />
    <binding protocol="http" bindingInformation="*:port:ip" />
    <binding protocol="https" bindingInformation="*:port:ip" />
  </bindings>
</site>
```
