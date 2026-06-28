---
title: Incident Response - Lessons Learned
date: 2026-06-28 00:06:00 +0200
categories: [Incident Response]
tags: [incident response, lessons learned]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective

> A look into the fifth phase of the Incident Response framework: Lessons Learned.

---

In this phase, we should sit down with the data gathered throughout the IR process, through the different phases such as **Preparation**, **Identification & Scoping**, **Containment and Threat Intel Creation**, and **Eradication, Remediation and Recovery**. The different actions taken, not taken, all should be evaluated, so that we may be able to learn from the experience, to do better in the future. 

Often, at this phase, we must create two documents: Technical Summary and Executive Summary, that summarizes all relevant findings for the appropriate audience. Executive Summary is for the stakeholders, and often they will want to know the impact on the business in the macro sense, rather than the technical aspects. 

It is also important to integrate the indicators from the **Spreadsheet of Doom (SoD)** to the SOC's detection mechanism. This is often done using vendor-agnostic tools such as Sigma, which allows us to describe log events in a structured format. 



---


