---
title: AI System Reconnaissance
date: 2026-10-16 00:00:00 +0200
categories: [AI Security]
tags: [ai security, ai attack surface, mlflow, triton, enumeration, network mapping]
series: ""
series_order: 
---

> **Disclaimer**: This post contains my personal notes and methodology for the respective TryHackMe room. All the credits for the room and the lab environment go to the original creators at TryHackMe. **Flags are not revealed** to preserve the challenge for others. 
{: .prompt-info}

## Objective


> Discover AI infrastructure by scanning ML services, frameworks, and extracting metadata from APIs.

---

## Scenario

> You are a security engineer at Cyphira, a mid-size fintech company that deployed several AI-powered features over the past year without a security review. Your CISO read a threat report about attackers hijacking exposed AI infrastructure and wants answers: which AI systems do we have, what they expose, and how an attacker could find them? Your job is not to break anything. It is to find everything.

## Practical Guide: AI Infrastructure Reconnaissance and Enumeration

### 1. Architectural Overview and Target Surface

Production AI and Machine Learning (AI/ML) infrastructures introduce specialized stacks for tracking, training, model registry, orchestration, and serving. Mapping these services requires targeting their specific ports, protocols, and standard paths.

#### Primary Service Fingerprints

| Platform / Tool | Layer | Default Ports | Primary API Paths / Signatures |
| :--- | :--- | :--- | :--- |
| **MLflow** | Experiment Tracking | `5000/TCP` | `/api/2.0/mlflow/runs/search`<br>`/api/2.0/mlflow/registered-models/search` |
| **Kubeflow** | Pipeline Orchestration | `3000/TCP`, `8080/TCP` | `/pipeline/apis/v1beta1/pipelines` |
| **Triton Inference Server** | Model Serving | `8000/TCP` (HTTP)<br>`8001/TCP` (gRPC)<br>`8002/TCP` (Metrics) | `/v2/health/ready`<br>`/v2/models`<br>`/metrics` |
| **TorchServe** | Model Serving | `8080/TCP` (Inference)<br>`8081/TCP` (Management)<br>`8082/TCP` (Metrics) | `/ping`<br>`/models`<br>`/models/{model_name}` |
| **vLLM / Ollama** | LLM Serving | `8000/TCP` (vLLM)<br>`11434/TCP` (Ollama) | `/v1/models`<br>`/api/tags` |
| **Ray Cluster** | Distributed Compute | `6379/TCP` (GCS)<br>`8265/TCP` (Dashboard)<br>`10001/TCP` (Client) | `/api/jobs/`<br>`/api/version` |
| **Jupyter Server** | Development | `8888/TCP` | `/api/kernels`<br>`/lab` |

---

### 2. Reconnaissance Workflow & Execution

#### Step 1: Network Discovery and Service Scanning

Use `nmap` to discover exposed ML framework ports across target ranges.

```bash
# Scan for common AI/ML tracking, serving, and orchestration ports
nmap -p 5000,6379,8000,8001,8002,8080,8081,8082,8265,8500,8501,8888,11434 \
  -sV -sC -Pn -iL targets.txt -oA ai_infrastructure_discovery
```

#### Step 2: Endpoint Fingerprinting and Path Enumeration

When web ports (`8000`, `8080`, `5000`, `8265`) are identified, probe for AI-specific routes using HTTP path enumeration tools.

##### Wordlist Generation (`ai_paths.txt`)
```text
/v2/health/ready
/v2/models
/v2/repository/index
/v1/models
/api/tags
/metrics
/api/2.0/mlflow/runs/search
/api/2.0/mlflow/registered-models/search
/api/version
/api/jobs/
/ping
/models
```

##### Executing Path Enumeration via `ffuf`
```bash
ffuf -u http://<TARGET_IP>:<PORT>/FUZZ -w ai_paths.txt -mc 200,401,403,500 -of json -o ffuf_ai_results.json
```

---

### 3. Extracting Model and Infrastructure Metadata

#### A. Extracting MLflow Experiment Metadata

If MLflow (port `5000`) is exposed without authentication, extract execution parameters, git branches, user details, and backend storage locations:

```bash
# Enumerate all registered models
curl -s -X GET "http://<TARGET_IP>:5000/api/2.0/mlflow/registered-models/search" | jq .

# Search for execution runs to uncover artifact locations (e.g., S3 buckets, local file paths)
curl -s -X POST "http://<TARGET_IP>:5000/api/2.0/mlflow/runs/search" \
  -H "Content-Type: application/json" \
  -d '{"experiment_ids": ["0"], "max_results": 10}' | jq .
```

![AI Enumeration]({{ "assets/img/ai_enum.png" | relative_url}})

![AI Enumeration 2, mlflow search]({{ "assets/img/ai_enum_2.png" | relative_url}})

#### B. Fingerprinting Triton Inference Server

Triton exposes model configurations, framework backends, and tensor shapes over its HTTP REST API (port `8000`).

```bash
# Check server health and get server version details
curl -s -i "http://<TARGET_IP>:8000/v2/health/ready"

# List loaded models and current status
curl -s -X POST "http://<TARGET_IP>:8000/v2/repository/index" | jq .

# Get specific model architecture and backend configuration
curl -s "http://<TARGET_IP>:8000/v2/models/<MODEL_NAME>/config" | jq .
```

#### C. Extracting Info from OpenAI-Compatible LLM Endpoints (vLLM / LocalAI)

```bash
# Enumerate available base models and fine-tunes
curl -s "http://<TARGET_IP>:8000/v1/models" | jq .

# Query Ollama model tags
curl -s "http://<TARGET_IP>:11434/api/tags" | jq .
```

#### D. Mining Infrastructure Metrics (`/metrics`)

Prometheus endpoints often leak operational context, including hardware details, model names, and pipeline metrics.

```bash
# Scrape Prometheus metrics for model titles and hardware parameters
curl -s "http://<TARGET_IP>:8002/metrics" | grep -E "(model|gpu|inference|memory)"
```


---

> The different services **constantly talk to each other**. This means, any platform misconfiguration, can lead to a domino effect. And it goes without saying, that the attack surface has increased
{: .prompt-danger}

---

### 4. Detection and Telemetry Signatures

Monitoring AI reconnaissance involves tracking specific web request patterns, endpoint access anomalies, and traffic directed at administrative ports.

#### Snort / Suricata Detection Rules

```text
# Detect unauthenticated access attempts to MLflow REST API
alert http $EXTERNAL_NET any -> $HOME_NET 5000 (msg:"AI-RECON MLflow API Search Request"; flow:to_server,established; http.uri; content:"/api/2.0/mlflow/"; sid:1000001; rev:1;)

# Detect Triton Inference Server model index queries
alert http $EXTERNAL_NET any -> $HOME_NET 8000 (msg:"AI-RECON Triton Model Repository Index Query"; flow:to_server,established; http.uri; content:"/v2/repository/index"; sid:1000002; rev:1;)

# Detect probing of Ray Cluster API
alert http $EXTERNAL_NET any -> $HOME_NET 8265 (msg:"AI-RECON Ray Cluster API Access"; flow:to_server,established; http.uri; content:"/api/jobs"; sid:1000003; rev:1;)
```

#### Log Analysis Queries (Elasticsearch / Splunk)

To search HTTP access logs for AI-focused scanning activity:

```text
# Splunk Query: Identify high-frequency probing of ML paths
index=web_logs (uri_path="*/v2/models*" OR uri_path="*/api/2.0/mlflow/*" OR uri_path="*/v1/models*" OR uri_path="*/metrics*")
| stats count dc(uri_path) as unique_paths by src_ip
| where count > 20 AND unique_paths > 3
```

---

> MITRE ATLAS, modelled after the ATT&CK framework, for AI specifically, covers threats to AI and ML in an organized manner, and provides a comprehensive vocabulary and structured knowledge. Mapping the findings to ATLAS techniques will give the reader more technical context. 
{: .prompt-tip}


---

## Structured Reconnaissance Methodology

**Passive Reconnaissance** -> **Active Scanning** -> **API Fingerprinting** -> **Metadata Extraction** -> **Supply Chain Review**

Here is the structured table mapping the tools, platforms, and techniques to their descriptions and reconnaissance phases:

| Tool / Platform | Description | Phase |
| --- | --- | --- |
| **Shodan / Censys / FOFA** | Internet-wide search for AI service banners | Phase 1 |
| **GitHub search (dorks)** | Find leaked credentials and configs in public repos | Phase 1 |
| **Nmap (with NSE scripts)** | Port discovery and service version detection | Phase 2 |
| **grpcurl** | Interact with gRPC services, dump protobuf schemas if reflection is enabled | Phase 2 |
| **ffuf / feroxbuster** | Directory brute-forcing with AI-specific wordlists | Phase 2, 3 |
| **curl** | Manual HTTP probing, header analysis, error triggering | Phase 3, 4 |
| **MLOKit (IBM X-Force Red)** | Automated MLflow enumeration and model exfiltration | Phase 4 |
| **Nuclei (ProjectDiscovery)** | Template-based scanning for known AI service misconfigurations | Phase 2, 3 |
| **Agrus Scanner** | Purpose-built shadow AI detection with 50+ AI-specific probes across all 65,535 ports | Phase 2 |

---
