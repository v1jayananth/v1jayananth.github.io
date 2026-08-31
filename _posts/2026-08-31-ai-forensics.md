---
title: AI Forensics
date: 2026-08-31 00:00:00 +0200
categories: [AI Security]
tags: [ai security, forensics, ai tools, dfir]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Explore AI DFIR and learn how it boosts your investigation capabilities.

---

## DFIR, and the need for a boost from AI

Why would DFIR (Digital Forensics and Incident Response) require a boost from AI (or automation)? 

- Vast amounts of data to be processed. 
- **Identifying attacks is like searching for a needle in a haystack. The challenge is to recognize patterns and find deviations at a large scale.**
- Scalability across different types of infrastructures is possible with AI without the same proportional increase in workload. 

There are some AI solutions already implemented into well known tools such as Splunk, Elastic, Microsoft Defender, Crowdstrike Falcon etc. A brief list is provided below: 

- Splunk UEBA, Elastic ML: AI uses unsupervised learning techniques to learn baseline behaviour of users and systems, to identify deviations / anomalies as potential threats. 
- Splunk NLP, Microsoft Defender for O365: Transformer-based language models (e.g., BERT, RoBERTa) classify messages as phishing or benign based on tone, structure, and known attack patterns.
- VirusTotal ML integrations: AI systems analyse file metadata, code signatures, and sandbox behaviour to detect threats.
- Crowdstrike Falcon + Charlotte AI: AI analyses past alert data, analyst feedback, and incident outcomes to rank alerts by severity and relevance. This reduces noise and surfaces the most urgent issues first, saving analysts time, a valuable resource in DFIR.

However, there are limitations to deploying AI in DFIR. Inherently, **AI is non-deterministic. Digital Forensics on the other hand, is a field that demands consistent and repeatable results**. Moreover, to evaluate AI, we need to use accuracy, precision and recall, which can be misleading in isolation. 

---

## AI & DFIR

We will look at how AI/ML can be used across 4 key areas of DFIR: 

1. **Image and Video Forensics**
    - One way in which AI/ML can help in this area is through **CNN (Convolutional Neural Networks)**. A CNN is a type of neural network that automatically learns patterns in data using small filters commonly used for images. This can also work with other types of data such as audio, time series or text where sequential patterns are important. 
    - Examples include CNN-based Forgery Detection, Deepfake Detection, GANs (Generative Adversarial Networks). 
2. **Communication Analysis**
    - Involves processing and analyzing large volumes of text. 
    - Examples include Phishing email analysis, Social Media analysis (**Sentiment Analysis**). 
3. **Timeline Reconstruction and User Behaviour**
    - ML can help with this a lot, since this is a very labour-intensive and time-consuming task, especially when the data is **time-sequenced**. 
    - Examples include Automated Event Timeline Reconstruction and Anomaly Detection. 
4. **Malware Detection / Analysis**
    - ML can help with providing an edge over traditional signature-based analysis
    - Examples include ML boosting dynamic analysis, utilizing deep neural networks to classify a file as malicious or benign, etc. 

---

## AI Legal & Ethical Implications

One of the core issues facing the implementation of AI in digital forensics is the explainability of AI tools. Many AI models are “black boxes”, meaning they don’t readily explain how they came to a conclusion. 

AI systems can unintentionally introduce bias, raising ethical and legal concerns about fairness and due process. ML models are trained on historical data; if that data contains skewed representations or prejudices, the model’s output will reflect them.

> Another key aspect to worry about AI, is **accountability**. Who is accountable for the AI's outputs and actions? 
{: .prompt-danger}

---

## Digital Trail Case

We have to perform a forensic analysis and determine what happened. In this case, we have some support from AI/ML systems, which can help us process the data faster. 

But first, **isolation is key**. Let's enable the virtual environment before making any changes, by running `source /opt/dfir-env/bin/activate`

We run a custom python script that uses **scikit-learn** for efficient data mining, against the **auth.log** log file: `python3 /opt/dfir-lab/classify_logs.py /var/log/auth.log`

The output we receive: 

```
[SUSPICIOUS] Jan 15 03:00:01 tryhackme sshd[1833]: Failed password for invalid user admin from 192[.]168[.]0[.]100 port 22 ssh2
[SUSPICIOUS] Jan 15 03:01:02 tryhackme sshd[1833]: Accepted password for j.morgan from 192[.]168[.]0[.]100 port 22 ssh2
[SUSPICIOUS] Jan 15 03:15:00 tryhackme sshd[1833]: Accepted publickey for r.house from 192[.]168[.]0[.]100 port 22 ssh2
```

This indicates that the attacker has indeed gained access. Let's run another python script provided to us, that scans **high-priority directories** for changes: `python3 /opt/dfir-lab/file_anomalies.py`

Output: 

```
[*] Scanning paths:
    - /opt/robbco
    - /tmp
    - /dev/shm
    - /usr/local/bin

[!] Supervised AI-flagged suspicious files:
                                                 path  ...   entropy
0                    /opt/robbco/sys/boot_monitor.log  ...  4.535458
1   /opt/robbco/engineering/MFBootAgent/mfboot_main.c  ...  5.122591
2           /opt/robbco/firmware/RETROS_BIOS/core.asm  ...  4.358886
3                               /tmp/dock-replace.log  ...  4.811436
4                                         /tmp/.syncd  ...  4.492156
5                                             /tmp/.x  ...  4.411602
6                               /tmp/invoice_dump.txt  ...  4.653766
10                   /dev/shm/.core_dump_2025.tgz.enc  ...  5.766267
14                              /usr/local/bin/sysmon  ...  4.411602

[9 rows x 3 columns]
```

`invoice_dump.txt` reveals further files, since it seems to be a data dump of sorts. Such as `/home/j.morgan/Documents/Invoices/invoice_Q1_2075.ods`. This file seems to be a **phishing email attachment** and contains macro code that sends the SSH keys to an external server. 

Going through `j.morgan`'s bash_history file, we find the command: `sudo nano /home/r.house/.ssh/authorized_keys` being executed. Attacker plants an SSH key here, and logs in as `r.house` using it. 

The suspicious files `/usr/local/bin/sysmon` and `/opt/robbco/sys/boot_monitor.log` indicate that the attacker used these files to plant persistence mechanisms. 

`/dev/shm/.core_dump_2025.tgz.enc` is an archive used to steal the proprietary source code of RobbCo. 

---


