---
title: Incident Response - Identification & Scoping
date: 2026-06-28 00:02:00 +0200
categories: [Incident Response]
tags: [incident response, identification, scoping]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> A look into the second phase of Incident Response: Identification & Scoping

---

This critical phase combines the technical detection of potential security incidents with the inherent human capacity to recognise and report them. The speed at which an organization can spot an incident **directly correlates** with the pace of response, potentially **limiting the damage** and shortening recovery time.

Scoping involves grasping the extent of a security incident, including:

- which systems are affected
- what data is at risk
- how the incident affects the organizations. The full impact of it. 


---

## Identification: Case Example

We have an email that says there is a potential security incident, possibly involving a phishing scam. Full details of it are included in the email, and we have to navigate through this, identifying key details. 

alex[.]swift[@]swiftspend[.]finance is not a real email, but alexander[.]swift[@]swiftspend[.]finance is. The former sent the phishing email. 

`Ticket#2023012398704232 - WKSTN-02.swiftspend.thm - IP 172[.]16[.]1[.]151`

```
Requesting Exchange Server logs and Message Trace pertaining to the following emails:
•	Ascot, Michael <michael[.]ascot[@]swiftspend[.]finance>
•	Swift, Alex <alex[.]swift[@]swiftspend[.]finance>

Additionally, can you retrieve the Web Proxy logs for the following machine:
•	WKSTN-02 (172[.]16[.]1[.]151)
```

Further emails sugggest that SPF, DKIM & DMARC hasn't been configured properly. So an attacker was probably able to spoof an email from the swiftspend.finance domain. 

EDR on the host WKSTN-02 shows that it isn't compromised with malware. 


---

```
Ticket#2023012398704233 - WKSTN-01.swiftspend.thm - IP 172[.]16[.]1[.]150

I received an email that redirects me to hxxps://kennaroads[.]buzz/data/Update365/office365/40e7baa2f826a57fcf04e5202526f8bd/?email=zoe[.]duncan[@]swiftspend[.]finance&error and asks me to login to Office 365.

```

---

## Scoping: Understanding the extent of the incident

Two main tools that follow, are indispensable tool when it comes to Scoping the extent of a security incident. The more information, and the higher quality, the better the following phases of incident response will be. 

### Asset Inventory

A crucial tool in the incident response process. Comprehensive list of all the organizations' assets. 

A simple example is as follows: 

![Image]({{ "assets/img/identification_1.png" | relative_url}})

---

### The Spreadsheet of Doom (SoD)

A consolidated, organized source of information about known threats. Serves as a **single reference point**. 

Each row in this spreadsheet represents a unique threat identifier or **Indicator of Compromise (IoC)**. 

The headers include: Indicator Type, Indicator, Threat Type and Source of the information. 

---

Going back to our case example

The main phishing email provides us with the phishing URL used: `hxxps://b24b-158-62-19-6[.]ngrok-free[.]app/`

---

## Intelligence-Driven Feedback Loop

This sort of approach encourages a proactive and dynamic method towards incident response. Facilitates ongoing exchange of information, enabling organizations to respond to security incidents efficiently. 

---

From john, the email says: 

```
As Oliver and I suspected, Mike received a spoofed email and this is indeed related to our mail server's security.

Received: from emkei.cz (89[.]187[.]129[.]25) by MAILSRV-01.swiftspendfinancial.thm
 (172[.]16[.]1[.]15) with Microsoft SMTP Server id 15[.]2[.]1118[.]7 via Frontend
 Transport; Thu, 13 Jul 2023 13:57:02 +0000
```

A further domain can be included in the Spreadsheet of Doom. 

And, alexander[.]swift[@]swiftspend[.]finance, also received an email from mike.ascot, with the same subject and phishing link. 

The emails show this message trace, which gives additional IoCs: 

```
Received,SenderAddress,RecipientAddress,Subject,Status
2023-07-12T16:04:59.23145827Z,sales[.]tal0nix[@]gmail[.]com,alexander[.]swift[@]swiftspend[.]finance,(No subject),Delivered
2023-07-12T16:04:59.23145827Z,sales[.]tal0nix[@]gmail[.]com,michael[.]ascot[@]swiftspend[.]finance,(No subject),Delivered
2023-07-13T13:57:02.58642720Z,alex[.]swift[@]swiftspend[.]finance,michael[.]ascot[@]swiftspend[.]finance,Proposal From United Trust Company Limited,Delivered
2023-07-13T13:57:02.58642720Z,mike[.]ascot[@]swiftspend[.]finance,alexander[.]swift[@]swiftspend[.]finance,Proposal From United Trust Company Limited,Delivered
```


And we also find traces of the login: 

```
14,"July 13, 2023 X:XX:XX PM UTC","July 13, 2023 X:XX:XX PM UTC",michael[.]ascot[@]swiftspend[.]finance,No,b24b-158-62-19-6.ngrok-free.app/submit-login?username=michael[.]ascot[@]swiftspend[.]finance&password=Passw0rd!,Allowed,Post,200 - OK,"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114[.]0[.]0[.]0 Safari/537.36 Edg/114[.]0[.]1823[.]67",172[.]16[.]1[.]151,ZZZ.ZZZ.ZZZ.ZZZ,158[.]62[.]19[.]6,United Kingdom,General Browsing,General Browsing,General Surfing,Miscellaneous,Miscellaneous or Unknown,None,None,None,None,0,None,None,Default Department,ZZZ,ZZZ,b24b-158-62-19-6.ngrok-free.app
```

---


