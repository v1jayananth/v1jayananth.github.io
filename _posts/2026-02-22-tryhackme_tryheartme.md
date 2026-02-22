---
title: TryHeartMe - Web-based TryHackMe Challenge
date:  2026-02-22 00:00:00 +0100
categories: [TryHackMe Challenges]
tags: [tryhackme, challenges, web, jwt]
---

# Goal

The goal is to find the secret **ValenFlag**, which as most tryhackme flags go, should probably start with **THM**. 

# Initial recon

- Using `nmap`, we can see that two ports are open
  - Port `22`, default ssh
  - Port `5000`, which hosts a website

# Website

- The challenge provides a website at port `5000`
- You can browse around as a guest. There are many things to buy, and they require **credits**. 
- You can **login** and **signup**. 
- **Signing up** takes a email address and a password of minimum 6 characters. That's it. 
- Once you do that, while **inspecting** the page, with the tab set on **network** or **storage**/**cookies**, you will see a **jwt-token** with the variable name **tryheartme_jwt** being set. 
- As a user, you can log out and login, and nothing is revealed to be special here. **Mainly, you do not have any credits. Your credits are 0**. 
- Moreover, when you inspect the page, you can see the Server being used
  -Server: Werkzeug/3.0.1 Python/3.12.3, might be of interest? 

# Exploit, or one way to do it

- If you go to [jwt.io Website, to decode jwt](https://www.jwt.io), you can paste the jwt you received, and see the output
- There are 5 fields revealed:
  - email, the email of your account
  - **"role": "user"**
  - **"credits": 0**,
  - iat
  - theme
- Clearly, we have to do something with the role or the credits. 

> Time to load cyberchef :)
> {: .prompt-tip}

- With cyberchef, you have a recipe called **JWT-Sign**. Use it
- The default private signing key is **secret**, let's see if it works. 
- In the input, place the decoded json, change the role to **admin** or increase your credits
- Encode it

![CyberChef, JWT Sign]({{ "/assets/img/cyberchef_tryheartme.png" | relative_url }})

- Now you have a modified jwt, take it, and replace your original cookie with this (inspect page -> storage -> cookies), and reload the page
- You should now have 5000 credits (if you modified role to **admin**), and an option to buy the valenflag for 777. Since 5000 > 777 (obviously), just buy it and you will get the flag

# Vulnerability

**What went wrong here?**

> The developers of this vulnerable used a guessable, short, key **secret**. 
> {: .prompt-danger}

## JSON Web Token

- Consists of three parts: **Header**, **Payload** and **Signature**. 
- The server doesn't hide data or encrypt it. It just encodes in base64. 
- However, **digital signatures** are used to ensure the data isn't tampered with. 
- The header and payload and signed with a private signing key. 

In our case, when we encoded JSON in CyberChef with private signing key "secret" and set the resulting jwt in the server as a cookie, it recalculates using it's own key as well. If the signatures don't match then it shouldn't work. But it does, because a weak, "easy-to-guess" private signing key "secret" was used. 

> In real life, one would use jwt crackers that brute force the private signing key using a wordlist or some pattern
> {: .prompt-tip}

> It is also important to note, that there may be other ways too. JWTs have many vulnerabilities
> {: .prompt-warning}

**Happy Hacking!**
