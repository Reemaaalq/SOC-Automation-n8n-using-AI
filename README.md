# SOC-Automation-n8n-using AI
<div align="center">

# 🛡️ SOC Automation Lab

### AI-Powered Security Operations Center — Automated Alert Triage Pipeline

[![Splunk](https://img.shields.io/badge/Splunk-Enterprise-FF5733?style=for-the-badge&logo=splunk&logoColor=white)](https://www.splunk.com)
[![n8n](https://img.shields.io/badge/n8n-Workflow_Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)](https://n8n.io)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--5-412991?style=for-the-badge&logo=openai&logoColor=white)](https://openai.com)
[![VirusTotal](https://img.shields.io/badge/VirusTotal-Threat_Intel-394EFF?style=for-the-badge&logo=virustotal&logoColor=white)](https://www.virustotal.com)
[![TheHive](https://img.shields.io/badge/TheHive-5.x-FFA500?style=for-the-badge)](https://thehive-project.org)
[![Slack](https://img.shields.io/badge/Slack-Notifications-4A154B?style=for-the-badge&logo=slack&logoColor=white)](https://slack.com)

[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen?style=flat-square)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-Welcome-brightgreen?style=flat-square)](CONTRIBUTING.md)
[![Maintained](https://img.shields.io/badge/Maintained-Yes-brightgreen?style=flat-square)]()
[![Platform](https://img.shields.io/badge/Platform-Linux%20%7C%20Docker-informational?style=flat-square)]()
[![MITRE ATT&CK](https://img.shields.io/badge/MITRE-ATT%26CK%20Mapped-red?style=flat-square)](https://attack.mitre.org)
[![Pipeline](https://img.shields.io/badge/Pipeline-Operational-success?style=flat-square)]()

<br/>

> **An enterprise-grade SOC automation pipeline** — Splunk detects threats and fires alerts, n8n orchestrates the response, GPT-5 analyses and enriches via VirusTotal, and findings are simultaneously delivered to **Slack** for analyst notification and **TheHive** for structured case creation. Zero manual Tier-1 triage required.

<br/>

[📖 Overview](#-project-overview) · [🏗️ Architecture](#-architecture) · [🚀 Quick Start](#-quick-start) · [⚙️ Configuration](#-configuration) · [📋 Workflow](#-workflow-walkthrough) · [📦 JSON Samples](#-json-samples) · [🗺️ Roadmap](#-roadmap)

</div>

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Key Features](#-key-features)
- [Architecture](#-architecture)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Repository Structure](#-repository-structure)
- [Configuration](#-configuration)
- [Workflow Walkthrough](#-workflow-walkthrough)
- [JSON Samples](#-json-samples)
- [Alert Severity Matrix](#-alert-severity-matrix)
- [MITRE ATT&CK Coverage](#-mitre-attck-coverage)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🔍 Project Overview

Security teams are overwhelmed by alert volume. Analysts burn the majority of their shift on **Tier-1 triage** — manually parsing, contextualising, and routing alerts — leaving little time for real threat hunting.

This project solves that with a **5-node automated SOC pipeline** built entirely in n8n:

1. **Splunk** detects a threat via a correlation search and fires a JSON webhook
2. **n8n** receives the alert through a Webhook trigger node
3. **GPT-5** analyses the alert and calls **VirusTotal** in real time for IOC enrichment
4. **Slack** receives a rich, AI-generated analyst briefing — severity, summary, recommended actions
5. **TheHive** gets a fully structured case created automatically — no human touch required

From raw Splunk alert to Slack notification and TheHive case in **under 60 seconds**.

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🤖 **GPT-5 AI Analysis** | OpenAI GPT-5 generates analyst-grade triage summaries and action recommendations per alert |
| 🔍 **VirusTotal Enrichment** | Live IOC lookups (IPs, hashes, domains) via VirusTotal API called as a GPT-5 tool |
| 🔄 **Parallel Fan-Out** | Slack notification and TheHive case created simultaneously from a single GPT-5 response |
| 📊 **Splunk Integration** | Correlation search webhook fires directly into the n8n pipeline |
| 🔔 **Rich Slack Alerts** | Block Kit formatted notifications with severity colour, AI summary, and TheHive case link |
| 📝 **Auto Case Management** | TheHive cases created with title, description, severity, TLP, MITRE tags, and observables |
| 🛡️ **MITRE ATT&CK Mapped** | Every alert tagged to the relevant ATT&CK Tactic and Technique |
| ⚡ **Sub-60s Response** | End-to-end from Splunk alert to Slack + TheHive in under a minute |
| 🐳 **Dockerised Stack** | n8n and TheHive deployable with a single `docker compose up` |

---

## 🏗️ Architecture

### The 5-Node n8n Workflow

```
┌──────────────────────────────────────────────────────────────────────┐
│                         n8n TRIAGE WORKFLOW                          │
│                                                                      │
│                                                                      │
│  ┌────────────┐   POST    ┌──────────────────────────────────────┐  │
│  │            │  1 item   │                                      │  │
│  │  Webhook   │──────────▶│       Message a Model (GPT-5)   Tools│──┼──▶ ┌─────────────────┐
│  │            │           │                                      │  │    │   HTTP Request  │
│  └────────────┘           └────────────────┬─────────────────────┘  │    │  (VirusTotal    │
│                                            │  Response Text         │    │   API lookup)   │
│                                            │                        │    └─────────────────┘
│                              ┌─────────────┴──────────────┐         │
│                              │                            │         │
│                              ▼                            ▼         │
│                  ┌─────────────────────┐    ┌────────────────────┐  │
│                  │  Send a Message     │    │  Create a Case     │  │
│                  │  (Slack)            │    │  (TheHive)         │  │
│                  └─────────────────────┘    └────────────────────┘  │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```
<img width="1161" height="537" alt="image" src="https://github.com/user-attachments/assets/0df97ba3-6420-48ff-b7f9-29cb424759d3" />


### End-to-End Data Flow

```
Splunk Correlation Search Fires
           │
           │  HTTP POST (JSON alert payload)
           ▼
  ① Webhook Node
     Receives the raw Splunk alert body
     and passes it downstream to GPT-5
           │
           ▼
  ② Message a Model — GPT-5
     System prompt + alert JSON sent to OpenAI GPT-5
     GPT-5 calls VirusTotal tool for live IOC enrichment
           │
           ├── Tool call ──▶  ③ HTTP Request (VirusTotal)
           │                      IP / hash / domain reputation
           │                      Score, malware families, community votes
           │◀──────────────────── Returns enrichment JSON
           │
           │  Structured triage output:
           │  summary · severity · MITRE tags · VT findings · actions
           │
      ┌────┴────────────────────────┐
      │                             │   (parallel execution)
      ▼                             ▼
  ④ Send a Message              ⑤ Create a Case
     Slack Block Kit alert          TheHive case
     • Severity + colour            • Title / Description
     • GPT-5 AI summary             • Severity / TLP / PAP
     • VirusTotal findings          • MITRE ATT&CK tags
     • Recommended actions          • Observable: src IP / hash
     • Link to TheHive case         • Analyst task auto-created
```
<img width="1872" height="906" alt="image" src="https://github.com/user-attachments/assets/e7ed7ed4-e712-47a4-95b3-ba724bad2dce" />
> alerts on splunk
<img width="1845" height="737" alt="image" src="https://github.com/user-attachments/assets/4bb79ced-426f-4d03-b1af-42c976cc39c0" />
> Findings sent to Slack
<img width="1912" height="871" alt="image" src="https://github.com/user-attachments/assets/0da15da9-28a6-4d88-8989-d1d27d827363" />
> Case created in TheHive
### Infrastructure Diagram

```
  ┌─────────────────────────────────────────┐
  │            SPLUNK ENTERPRISE            │
  │                                         │
  │  Correlation Search  ──▶  Alert Action  │
  │  (rule match)              (webhook)    │
  └─────────────────────┬───────────────────┘
                        │  POST /webhook/splunk-alert
                        ▼
  ┌─────────────────────────────────────────┐
  │                  n8n                    │
  │   Webhook → GPT-5 → Slack + TheHive     │
  └──────────┬──────────────────┬───────────┘
             │                  │
             ▼                  ▼
  ┌──────────────────┐  ┌───────────────────┐
  │      Slack       │  │     TheHive 5     │
  │  #soc-alerts     │  │  Case Management  │
  │  Block Kit alert │  │  Analyst Workflow │
  └──────────────────┘  └───────────────────┘
             ▲
  ┌──────────┴───────────┐
  │  OpenAI GPT-5        │
  │  + VirusTotal (tool) │
  └──────────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Role | Version |
|---|---|---|
| **Splunk Enterprise** | SIEM — log ingestion, correlation searches, alert webhook | 9.x |
| **n8n** | Workflow orchestration — the automation glue layer | 1.x (self-hosted) |
| **OpenAI GPT-5** | AI triage — alert analysis, severity scoring, action recommendations | GPT-5 |
| **VirusTotal** | Threat intelligence enrichment — IP, hash, and domain reputation lookups | API v3 |
| **Slack** | Real-time analyst notification via Block Kit formatted messages | Bot API v2 |
| **TheHive** | Case management and structured incident response platform | 5.x |
| **Docker / Compose** | Container orchestration for n8n and TheHive | 27.x |

---

## ✅ Prerequisites

- **OS:** Ubuntu 22.04 LTS / Debian 12 *(or Docker Desktop on macOS/Windows)*
- **RAM:** 8 GB minimum · 16 GB recommended
- **Disk:** 40 GB free
- **Docker Engine:** 24.0+ with Compose v2
- **Splunk Enterprise** installed and licensed (or Splunk Free trial)

**Required API Keys:**

| Service | Where to obtain | Free tier |
|---|---|---|
| OpenAI API Key | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) | GPT-5 access required |
| VirusTotal API Key | [virustotal.com](https://www.virustotal.com/gui/join-us) | ✅ 500 req/day |
| Slack Bot Token | [api.slack.com/apps](https://api.slack.com/apps) → scopes: `chat:write` | ✅ Free |
| TheHive API Key | TheHive admin panel → Users → API Key | ✅ Self-hosted free |

---

## 🚀 Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/soc-automation-lab.git
cd soc-automation-lab

# 2. Configure environment variables
cp .env.example .env
nano .env          # Add your API keys (see Configuration section)

# 3. Start n8n and TheHive
docker compose up -d

# 4. Verify containers are healthy
docker compose ps

# 5. Import the n8n workflow
# Open http://localhost:5678
# Go to Menu → Import workflow → upload workflows/soc-triage-main.json

# 6. Configure Splunk alert webhook
# Splunk → Settings → Alert Actions → Webhook
# URL: http://<your-n8n-host>:5678/webhook/splunk-alert

# 7. Fire a test alert manually
curl -X POST http://localhost:5678/webhook/splunk-alert \
  -H "Content-Type: application/json" \
  -d @samples/test-alert-brute-force.json
```

> ✅ **Expected result:** Within ~30 seconds — Slack receives a GPT-5 triage alert with VirusTotal findings, and a new case appears in TheHive.

---

## 📁 Repository Structure

```
soc-automation-lab/
│
├── docker-compose.yml               # n8n + TheHive + Elasticsearch
├── .env.example                     # Environment variable template
├── README.md
│
├── splunk/
│   ├── savedsearches.conf           # Correlation search definitions
│   ├── alert_actions.conf           # Webhook alert action config
│   └── transforms.conf              # Field extraction lookups
│
├── n8n/
│   └── workflows/
│       └── soc-triage-main.json     # ★ Importable 5-node workflow
│
├── gpt/
│   └── prompts/
│       ├── system-prompt.txt        # GPT-5 system instruction
│       └── user-prompt-template.txt # Per-alert prompt template
│
├── thehive/
│   ├── config/
│   │   └── application.conf         # TheHive service config
│   └── scripts/
│       └── create_case.py           # Standalone Python TheHive helper
│
├── slack/
│   └── templates/
│       └── block-kit-alert.json     # Slack Block Kit message template
│
├── samples/
│   ├── test-alert-brute-force.json
│   ├── test-alert-malware-hash.json
│   └── test-alert-c2-beacon.json
│
└── docs/
    ├── ARCHITECTURE.md
    ├── WORKFLOW_GUIDE.md
    └── TROUBLESHOOTING.md
```

### Docker Compose

```yaml
# docker-compose.yml
version: "3.9"

services:

  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=${N8N_USER}
      - N8N_BASIC_AUTH_PASSWORD=${N8N_PASSWORD}
    volumes:
      - n8n-data:/home/node/.n8n

  thehive:
    image: strangebee/thehive:5.4
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - ./thehive/config/application.conf:/etc/thehive/application.conf
      - thehive-data:/opt/thp/thehive/db
    environment:
      - JVM_OPTS=-Xms512m -Xmx2g
    depends_on:
      - elasticsearch

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.14.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms1g -Xmx1g
    volumes:
      - es-data:/usr/share/elasticsearch/data

volumes:
  n8n-data:
  thehive-data:
  es-data:
```

---

## ⚙️ Configuration

### `.env` File

```ini
# ── n8n ───────────────────────────────────────────────────────────────────
N8N_USER=admin
N8N_PASSWORD=ChangeMe123!

# ── OpenAI ────────────────────────────────────────────────────────────────
OPENAI_API_KEY=sk-your-openai-api-key-here
OPENAI_MODEL=gpt-5

# ── VirusTotal ────────────────────────────────────────────────────────────
VIRUSTOTAL_API_KEY=your_virustotal_api_key_here
VIRUSTOTAL_BASE_URL=https://www.virustotal.com/api/v3

# ── TheHive ───────────────────────────────────────────────────────────────
THEHIVE_URL=http://thehive:9000
THEHIVE_API_KEY=your_thehive_api_key_here
THEHIVE_DEFAULT_TLP=2          # 0=WHITE  1=GREEN  2=AMBER  3=RED
THEHIVE_DEFAULT_PAP=2

# ── Slack ─────────────────────────────────────────────────────────────────
SLACK_BOT_TOKEN=xoxb-your-slack-bot-token
SLACK_CHANNEL_ID=C0XXXXXXXXX   # #soc-alerts channel ID

# ── Splunk ────────────────────────────────────────────────────────────────
SPLUNK_WEBHOOK_SECRET=your_shared_secret_here
```

### GPT-5 System Prompt

```
# gpt/prompts/system-prompt.txt

Act as a  Teir 1 SOC Analyst - summarize Cleary when provided with a security alert or incident details - Perform the following steps:

summarize alert (including indicators of Compromise, logs, or metadata), Perform the following steps:

Summarize the alert - Provide a clear summary of what triggered the alert, which systems/users are affected, and the nature of the activity (e.g., suspicious login, malware detection, lateral movement)

Enrich with threat intelligence - correlate any IOCs (IP address, domains, hashes) , with known threat intel sources, scan the hash via virustotal . Highlight if the indicators are associated with known malware or threat actors.

assess severity - Based on MITRE ATTACK mapping, identify tactics /techniques and provide an initial severity rating (Low, Medium, High, Critical), Finally send Findings to theHive and create a case for investigation if neccasary 

Recommend Next Actions- Suggest investigation steps and potential containment actions.

