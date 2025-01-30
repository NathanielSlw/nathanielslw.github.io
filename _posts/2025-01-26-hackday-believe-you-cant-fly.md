---
title: HackDay 2025 -I Believe You Can't Fly
date: 2025-01-26
categories:
  - CTFs
  - HackDay 2025
tags:
  - forensic
---
## Description 

Category: Forensic 

> In the world of Featherstone Airways, where airships dominate the skies and steam-powered engines hum in harmony with the winds, disaster strikes aboard Flight 404. A cacophony of alarms blares through the cabin, and an automated voice pierces the tension:  "Alert: Navigation systems compromised. Manual override unavailable."
>
> The captain, drenched in sweat, confesses that the ship’s intricate steam-core systems have been infiltrated by rogue machinists. The autopilot is spewing erratic commands, and the controls have been rendered useless. Amid the chaos, your gaze falls upon a forgotten device—a mechanical tablet left behind by the ship’s chief engineer. Its dimly glowing screen is your only hope to uncover the secrets of this sabotage and reclaim control of the vessel.
> 
> The tablet appears to hold critical files containing traces of the hackers’ interference. To restore the autopilot and prevent the airship from plunging into the abyss, you must uncover the password hidden within these files. As the last passenger with a keen mind for cyber-steam security, it falls to you to analyze these files, piece together the password, and save the airship before it’s too late. Time is of the essence, and the lives of everyone aboard rest in your hands. Will you rise to the challenge and prove yourself the hero of the skies?

3 Files : 
* `plane_logs.txt`
* `say_hi.jpg`
* `whoami.jpg`

[Link of the event](https://ctftime.org/event/2615)
## Solve the challenge

In the file `plane_logs.txt`:

1. **Encoded Messages:**    
    - `ercbafr` (ROT13) → **"reponse"**
    - `c29sdXRpb24=` (Base64) → **"solution"**
    - `636c6566` (Hexadecimal) → **"clef"**
    - `0110001011111011011000110110100001100101` (Binary) → **"bûche"**
    - `KDFNGD~dPbCbiU6H` (yet to decode).

2. **Key Clues in the File:**
    - **Clue 1:** "Sometimes the answer is just three steps ahead." ✅  
        _Solution:_ A Caesar Cipher with a shift of 3.
    - **Clue 2:** "The most secure password is often the simplest one." ✅  
        _Solution:_ The password for the file `say_hi.jpg` is **"securepassword"**.

### Step 1: Decoding the Encrypted String

The encoded message `KDFNGD~dPbCbiU6H` can be decrypted using a Caesar Cipher with a shift of 3. 

Applying the Caesar Decipher using the ASCII table reveals :
```
KDFNGD~dPbCbiU6H → HACKDAY{aM_@_fR3E
```

### Step 2: Extracting the Hidden Data from `say_hi.jpg`

Using **StegSeek**, we analyze the `say_hi.jpg` file:
```
stegseek say_hi.jpg  
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek  

[i] Found passphrase: "securepassword"  
[i] Original filename: "shh.txt".  
[i] Extracting to "say_hi.jpg.out".  
```

The extraction reveals the final part of the flag:
```
-@LbaRT0s5}  
```

### Final Flag

Combining both parts, we get the full flag:  
**`HACKDAY{aM_@_fR3E-@LbaRT0s5}`**


