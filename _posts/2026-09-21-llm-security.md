---
title: LLM Security
date: 2026-09-21 00:00:00 +0200
categories: [AI Security]
tags: [ai security, llm security, prompt injection, memory poisoning, phishing]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Understand LLMs as an attack surface and get an overview of LLM security threats.

---

## Data-based threats

LLMs are fundamentally data-driven. 

Sometimes LLMs can inadvertently leak data by design because they memorize and repeat patterns from their training data. The usual way to extract this would be to generate large amounts of text from an LLM and analyze the outputs to identify sequences that indicate patterns. 

Another type of attack would be membership inference: ask the LLM if it ever recorded a specific data sample.

And finally, prompt leakage. Including **system prompt** leakage, which can reveal what the LLM is explicitly is allowed / denied to do. Might allow to circumvent those guards. 

---

## Model-based threats

> Spoiler Alert! Adding an LLM to your ecosystem will also add **model-based threats** to your ecosystem, because of the model being added to your ecosystem. 
{: .prompt-danger}

These attacks may expose **Intellectual Property** or other sensitive data, like the **training data** of the model. 


### Model extraction

Copying the model's parameters without authorization. 

An attacker can do this if they can communicate with the LLM through an API, and send a **large number of prompts**, storing the responses with those prompts as input-output pairs. These pairs can then be used to **train another model**, imitating the target model's behaviour. 

### Model Inversion

Exploits a model's output to reveal information about its training data. By storing many responses from the LLM, the attacker can understand and gather insights about the models' training data. 

**The model is treated as a source of information.**

---

## System based threats

**LLMs process all input (system instructions, user prompts) as a single concatenated context item, without any boundaries.**

This aspect enables **prompt injection, memory poisoning** and more. 

### Prompt injection

The manipulation of the model's input context to override or subvert its intended behaviour. Crucially, the model does not have a mechanism to distinguish between sources (input sources from the API, external database sources etc). 

### Context overflow

A context window, is essentially the amount of tokens the model can handle at once. If the number of tokens exceeds the limit, the model may get overwhelmed, causing delays or denial of service. The model may just forget some aspects previously mentioned, because the context is full. 

The attacker just provides extremely long inputs to overwhelm the model. 

### Memory poisoning

LLMs maintain persistent conversation. Meaning, there is a memory that attackers can poison. This can be done by **gradually injecting malicious or misleading information, influencing later outputs.**

---

## User-based threats

Users can be tricked, since AI is a new emerging technology, attackers can use this new attack surface, to target users themselves. 

**LLM Powered Social Engineering** is a rapidly emerging threat. Scams are starting to be more convincing than ever before, and AI makes it easy to modify it **quickly** based on the target, at a pace not possible before. There is also the added threat of AI enabling deepfakes, and voice alterations. 

Another huge threat is **trust**. **AI models sound trustworthy**, but their responses can be from anywhere, and without the right sources, people could unknowningly install malware or be tricked in any sort of manner. 

---


