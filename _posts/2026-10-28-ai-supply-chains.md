---
title: AI Supply Chains
date: 2026-10-28 00:00:00 +0100
categories: [AI Security]
tags: [ai security, ai supply chains, dependencies, model registry, ai risks]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Explore how AI's dependency on external models, datasets, and packages creates attack surfaces.


## Introduction

Artificial Intelligence and Machine Learning supply chains introduce unique attack vectors that traditional software supply chain security tools fail to cover. While traditional software relies primarily on source code repositories, package managers, and build pipelines, the AI ecosystem introduces **non-deterministic artifacts: trained model weights, serialized execution logic, specialized datasets, and deep transitive framework dependencies**.

---

## Core Concepts of the AI Supply Chain

### Supply Chain and Transitive Dependencies

A supply chain is an interconnected network of internal and external dependencies required to develop, build, test, and deploy an application. Transitive dependencies occur when direct dependencies rely on other packages. For instance, if a machine learning application imports `transformers`, `transformers` relies on `torch`, `torch` relies on `numpy`, and `numpy` relies on compiled C extensions. If an attacker publishes a malicious package with a name confusingly similar to a popular package (typosquatting) or performs dependency confusion on an internal package registry, pulling a direct dependency can transitively execute malicious installer scripts or inject arbitrary code into the runtime environment.

> When you call a model through an API, you are trusting the provider's entire pipeline: their training data curation, fine-tuning decisions, hosting security, and update practices.
{: .prompt-warning}


### AI Model Artifacts

Unlike traditional software binaries compiled from human-readable source code, trained machine learning models are statistical representations of data stored as multi-dimensional arrays of floating-point numbers (weights and biases). However, modern model formats frequently combine raw weights with executable code or serialization logic.

A model consists of:
- **Architecture Definition:** The code or graph blueprint describing how layers connect.
- **Model Weights:** The learned parameters (`.bin`, `.safetensors`, `.pt`, `.h5`).
- **Metadata / Configuration:** Tokenizers, vocabularies, and hyperparameters.

![Image]({{ "assets/img/ai_architecture.png" | relative_url}})


### Serialization Risks and SafeTensors

Many legacy formats (such as PyTorch `.pt`/`.pth` or pickle-based files) rely on Python's built-in `pickle` module. Pickle allows the serialization and deserialization of arbitrary Python objects. When a user runs `torch.load('model.pt')`, Python executes **arbitrary bytecode embedded inside the pickle payload**, achieving Remote Code Execution (RCE) instantly upon loading.

To mitigate this risk, HuggingFace introduced `SafeTensors`, a secure file format for storing tensors safely. It **prevents arbitrary code execution** because it contains only raw tensor data and a strict JSON header, completely disallowing executable Python objects.

---


## Transfer Learning and Fine-Tuning Mechanics

Transfer learning involves taking a pre-trained base model and adapting it to a downstream task by adjusting weights using a custom dataset. Parameter-Efficient Fine-Tuning (PEFT) techniques like **LoRA (Low-Rank Adaptation)** and **QLoRA (Quantized Low-Rank Adaptation)** freeze the original base model weights and inject trainable rank decomposition matrices.

LoRA and QLoRA only modify the small adapter weights, **leaving the vast majority of the base model untouched**. Consequently, if malicious behaviors or backdoors are deeply embedded in the base model weights, standard fine-tuning or LoRA training will not remove them. The model will retain its poisoned capabilities while successfully learning the new fine-tuned task.

---

## AI Supply Chain Components and Attack Surfaces

### Components

- **Models:** Pre-trained weights downloaded from public hubs.
- **Datasets:** Raw or processed text, image, and audio corpuses used for pre-training or alignment.
- **Frameworks:** Execution runtimes and libraries (`pytorch`, `tensorflow`, `jax`, `transformers`).
- **Dependencies:** Downstream Python packages, CUDA drivers, and C++ underlying libraries.

### Attack Surfaces Across 4 Layers

1. **Model Layer:** Exploiting insecure deserialization, altering model architectures to leak data, or embedding adversarial weights.
2. **Dependency Layer:** Typosquatting and dependency confusion targeting package managers handling framework wrappers.
3. **Data Layer:** Data poisoning attacks where malicious records are injected into open-source training datasets to force model misclassification or bias.
4. **Infrastructure Layer:** Compromising MLOps build pipelines, CI/CD runners, and container registries used to host model-serving containers.

![Image]({{ "assets/img/4_attack_layers.png" | relative_url}})


---

## Real Incidents

- **PyTorch Dependency Confusion (2022):** PyTorch published a CPU-only version of a package with a dependency namespace collision, allowing external packages to take precedence over internal packages, resulting in automatic arbitrary code execution.
- **HuggingFace Hub Vulnerabilities:** Multiple incidents where malicious models uploaded to public repositories used embedded Python scripts or pickle exploits to execute reverse shells when loaded by users.
- **Ultralytics Build Pipeline Compromise (2024):** Attackers compromised the build pipeline of popular YOLO repository dependencies, injecting malicious payloads into downstream user installations.
- **Solana Compromise (2024):** Supply chain vectors targeting AI/Web3 integration wrappers led to private key exfiltration via compromised utility packages.
- **NullifAI Scanner Evasion (2025):** Advanced techniques demonstrated where malicious payloads inside model metadata or custom layers evaded automated security scanners by obfuscating pickle bytecodes.

---

## Task

> You are a developer at a startup. Your team lead found a sentiment analysis model and wants to deploy it to production. Before you run `pip install` and `model.load()`, you need to evaluate whether it is safe to do so.

![Image]({{ "assets/img/supplychains_task_1.png" | relative_url}})

![Image]({{ "assets/img/supplychains_task_2.png" | relative_url}})

In contrast, a more legitimate model would look like: 

![Image]({{ "assets/img/supplychains_task_3.png" | relative_url}})

> It goes without saying that, while yes, more downloads, and a reputable organization can be trusted more than unverified organizations and under-review models, this is not a guarantee. It's always better to use other techniques like sandboxing and effective isolation from critical assets, and real technical tools to actually verify if the model is performing malicious actions or has vulnerabilities, before allowing it to be used in workflows. 
{: .prompt-info}


