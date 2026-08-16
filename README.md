# 🛡️ Edge Traffic Cloaker & Smart Router

<div align="center">

[![Type](https://img.shields.io/badge/Project-Architecture%20%26%20Reference%20Blueprint-blue?style=for-the-badge)](https://github.com/coutinhoicaro/edge-traffic-cloaker-router)
[![Runtime](https://img.shields.io/badge/Runtime-Cloudflare%20Workers-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/)
[![Latency](https://img.shields.io/badge/Latency-%3C%2015ms%20at%20Edge-2EA44F?style=for-the-badge)](https://github.com/coutinhoicaro/edge-traffic-cloaker-router)
[![Storage](https://img.shields.io/badge/State-Cloudflare%20KV-0052CC?style=for-the-badge&logo=redis&logoColor=white)](https://developers.cloudflare.com/kv/)
[![License](https://img.shields.io/badge/License-MIT-black?style=for-the-badge)](LICENSE)

<br>

**Serverless Edge Traffic Routing & Anti-Bot Protection**  
*A lightweight Cloudflare Worker that filters datacenter scrapers, blocks vulnerability probes, and routes clean traffic at the edge before it hits your origin server.*

</div>

---

## 📌 Overview

Automated bots, scrapers, and scanners consume origin server resources and generate useless traffic. 

This project demonstrates an **edge filtering architecture**:
* Runs on **Cloudflare Workers** (<15ms response time) across worldwide edge locations.
* Blocks datacenter IPs (like AWS, Hetzner, DigitalOcean) and automated scanning tools.
* Dynamically quarantines suspicious IPs in **Cloudflare KV** for 7 days.
* Silently redirects unwanted traffic to a safe white page while allowing legitimate users through.

---

## 🏗️ Traffic Decision Flow

```mermaid
flowchart TD
    subgraph EdgeEntry ["1. Request Ingress"]
        REQ["Incoming Request"] --> EXTRACT["Read IP, ASN, Country & Path"]
    end

    subgraph SecurityChecks ["2. Filtering Rules"]
        EXTRACT --> WL{"Whitelisted IP?"}
        WL -- "Yes" --> PASS["Forward to Target Application"]
        
        WL -- "No" --> Q{"In KV Quarantine?"}
        Q -- "Yes" --> REDIR["Redirect to Safe White Page"]
        
        Q -- "No" --> ASN{"Datacenter / Scraper ASN?"}
        ASN -- "Yes" --> REDIR
        
        ASN -- "No" --> PROBE{"Vulnerability Path (.env, wp-admin)?"}
        PROBE -- "Yes" --> BAN["Add to KV Quarantine (7 Days)"] --> REDIR
        
        PROBE -- "No" --> SIG{"Valid Query Key?"}
        SIG -- "No" --> REDIR
        SIG -- "Yes" --> PASS
    end

    style WL fill:#1e293b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style Q fill:#1e293b,stroke:#ef4444,stroke-width:2px,color:#fff
    style PROBE fill:#1e293b,stroke:#f59e0b,stroke-width:2px,color:#fff
    style PASS fill:#1e293b,stroke:#10b981,stroke-width:2px,color:#fff
```

---

## 🛡️ What It Filters

* **Datacenter Scrapers:** Instantly identifies traffic originating from server providers (AWS, OVH, Hetzner, DigitalOcean) using ASN checks.
* **Vulnerability Scanners:** Flags requests attempting to access sensitive paths like `/.env`, `/wp-login.php`, or `/admin` and puts the IP in temporary quarantine.
* **Non-Blocking Telemetry:** Uses `ctx.waitUntil` so security alert webhooks are sent in the background without slowing down real user responses.

---

## 📂 Repository Structure

```
edge-traffic-cloaker-router/
├── src/
│   └── worker.js           # Cloudflare Worker Filtering Logic & KV Handlers
├── package.json            # Node.js Config
├── wrangler.toml           # Worker Configuration Sample
├── LICENSE                 # MIT License
└── README.md               # Architecture Overview
```

> **Note:** This repository is an **architectural reference implementation**. Production IP lists, domain configurations, and webhook secrets are managed in private environment variables.

---

## 📄 License
Distributed under the **MIT License**. See `LICENSE` for details.
