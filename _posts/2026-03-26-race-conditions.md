---
title: Race Conditions in Web Applications
date: 2026-03-26 00:00:00 +0100
categories: [Penetration Testing]
tags: [penetration testing, tryhackme, race conditions, web applications]
series: ""
series_order: 
image:
  path: ./../assets/img/raceconditions_preview.png
  alt: Just a dumb picture of two cars "racing"
---

> **Disclaimer**: This post contains my personal notes on the relevant topic, but the credits for the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}


A **race condition** occurs when two or more operations compete to access shared state, with the outcome depending on timing. The core vulnerability is the **time-of-check to time-of-use (TOCTOU)** gap.

### Simple Example

Let's say you are calling a restaurant to reserve a table. You call the restaurant and one of the hosts picks up and confirms the table for you, saying it is free. At that same time, another call arrives, picked up by another host, and they also want the table you were just booking. At the time, the table is not confirmed yet, you both are on call with two different hosts, booking the same table. Both hosts will write down saying the table is reserved successfully to each of the callers, you and the other person. **So who reserved the table REALLY?**. Well, this is what a **Race condition** actually is. 

### Why Web Apps Are Vulnerable

- Stateless HTTP combined with shared mutable state (databases, caches)
- Horizontal scaling means no shared in-process locks across servers
- Application logic often uses separate SELECT + UPDATE queries
- Microservices amplify state divergence windows

### Common Exploit Patterns

| Attack | Mechanism |
|---|---|
| Double-spend | Redeem a coupon/gift card N times before any write commits |
| Limit bypass | Beat "one per user" checks with simultaneous requests |
| Privilege escalation | Downgrade account after check, before action |
| File upload race | Access file between upload and malware scan |

> Often tools such as Burp Suite's Turbo Intruder, is used to send many simulataneous HTTP requests, to redeem the same coupon or withdraw the same funds before a check happens. 
{: .prompt-info}

### Defenses

- **Atomic SQL** — `UPDATE ... WHERE balance >= amount` (single statement)
- **Pessimistic locking** — `SELECT ... FOR UPDATE`
- **Optimistic locking** — version column checked on write
- **Idempotency keys** — unique key per operation, rejected at DB constraint level
- **Distributed locks** — Redis `SET NX` to gate critical sections across servers

### More concrete examples in web applications

Where does this occur? Usually when **two threads** change values of one variable. More generally, race conditions occur wherever resources are **shared**. This happens a lot in web servers because:

1. Parallel execution: Without proper synchronization when serving many users at once, race conditions can lead to severe vulnerabilities. 
2. Database operations: Concurrent database operations, can introduce race conditions. 
3. Third-party libraries, APIs and services: Improper implementation here can affect every web application using it. 

> The solution overall is to enforce proper **locking** mechanisms, and transaction isolation. 
{: .prompt-info}

> Time windows are everything. During a time window from when we "do something", and for that to really happen in the application, how the program handles this and other processes, determines if there are race conditions or not. 
{: .prompt-danger}

## Exploiting Race Conditions

We are given a web application belonging to a mobile operator. We have username and password combo for two users, and our objective is to check if the web application is susceptible to a race condition vulnerability, in the following way: by transferring more credit than currently in our account. 


- User1: 07799991337
- Password: pass1234
- User2: 07113371111
- Password: pass1234

User 1 has 7.29$ as current balance. When trying to transfer 500$ to User2, we get the error of insufficient balance. 

Using burp suite, we can capture the request. Below we can see what happens when we send 2$ from User2 to User1. 

![Image 1]({{ "assets/img/raceconditions1.png" | relative_url}})

We can right click the request and **Send to Repeater**. 

The idea of the attack is as follows: 

1. Duplicate the current request as is on the Repeater tab **20 times**. (So that, you can deposit 20 times the amount of money per request) 
2. **Sending Requests in Parallel**, you basically instruct Burp Suite Repeater to send all those packets at once. 
3. And the idea is, sending requests in parallel at once, before the app has time to check the balances. It's not even race conditions 

To get the flag, we need to get 100$ in one of the accounts. So let's just deposit 200$. 10 per request (20 requests overall). 

![Image 2]({{ "assets/img/raceconditions2.png" | relative_url}})

Of course, the account should have 10$ in the beginning atleast, and that's all the prerequisites required. 

---

## Mitigations

### Synchronization Mechanisms

In a multi-threaded environment, synchronization ensures that only one thread can access a "critical section" of code at a time.

- **Mutexes (Mutual Exclusion)**: The most common form. If Thread A holds the mutex, Thread B must wait in line until Thread A releases it.
- **Distributed Locks**: Standard mutexes only work on a single server. Since modern web apps are usually load-balanced across multiple nodes, you need a distributed lock manager (like **Redis or ZooKeeper**) to ensure synchronization across your entire infrastructure.

---

### Atomic Operations

Atomic operations are "all or nothing." They are uninterruptible, meaning no other process can see the data in an intermediate, half-changed state.

- **Compare-and-Swap (CAS)**: A CPU-level or language-level instruction that updates a variable only if it matches a predicted value. If the value has changed since you last looked, the operation fails, forcing you to retry.
- **Application Level**: Many programming languages provide atomic counters (e.g., `AtomicInteger` in Java or `stdatomic.h` in C++). These are much faster than locks because they don't force threads to "sleep."

### Database Transactions

Strongest line of defense for web applications. But the right **Isolation Level** must be chosen: 

| Level | Mitigation | Trade-off |
| --- | --- | --- |
| Read Committed | Prevents "Dirty Reads" (reading uncommitted data). | Still vulnerable to most race conditions. |
| Repeatable Read | Ensures that if you read a row twice, the data is the same. | Can still suffer from "Phantom Reads." | 
| Serializable, The gold standard | Transactions are executed as if they were one after another. | High performance hit; can cause frequent timeouts/deadlocks. |

---

## Challenge Web App

Same type of scenario. Three users are given, and we have to get one of the users to have more than 1000$ without actually sending 1000$. Whatever the existing balance is, just send a multiple of that to reach 1000$. 

---

