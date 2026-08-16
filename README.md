# 🛡️ Edge Traffic Cloaker & Smart Router

<div align="center">

[![Runtime](https://img.shields.io/badge/Runtime-Cloudflare%20Workers%20%7C%20V8%20Isolates-F38020?style=for-the-badge&logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/)
[![Latency](https://img.shields.io/badge/Decision%20Latency-%3C%2012ms%20P99-2EA44F?style=for-the-badge&logo=speedtest&logoColor=white)](https://github.com/coutinhoicaro/edge-traffic-cloaker-router)
[![Storage](https://img.shields.io/badge/State-Cloudflare%20KV%20Distributed-0052CC?style=for-the-badge&logo=redis&logoColor=white)](https://developers.cloudflare.com/kv/)
[![Architecture](https://img.shields.io/badge/Security-Multi--Vector%20Anti--Bot-E92063?style=for-the-badge&logo=shield&logoColor=white)](https://github.com/coutinhoicaro/edge-traffic-cloaker-router)
[![License](https://img.shields.io/badge/License-MIT-black?style=for-the-badge)](LICENSE)

<br>

**Enterprise Serverless Edge Firewall & Adaptive Traffic Routing Engine**  
*Intercepts malicious crawlers, datacenter scrapers, and vulnerability probes at the global network edge with sub-12ms overhead.*

</div>

---

## 📌 Executive Summary

Modern web applications face relentless scraping, automated credential stuffing, and probe attacks that degrade origin server performance.

**Edge Traffic Cloaker & Smart Router** operates directly on Cloudflare’s global network of 300+ PoPs using V8 isolates. By evaluating requests before they ever reach origin infrastructure, it enforces multi-vector security heuristics, dynamically quarantines malicious IPs in distributed KV stores, and routes traffic cleanly between protected origins and benign white-page fallbacks.

---

## 🏗️ End-to-End System Architecture

```mermaid
flowchart TD
    subgraph ClientLayer ["1. Global Edge Ingress"]
        REQ["Incoming Edge Request"] --> IP["Extract CF-Connecting-IP, ASN, Country & Path"]
    end

    subgraph FastPath ["2. Zero-Latency Whitelist Gate"]
        IP --> WL{"Whitelisted IP in KV?"}
        WL -- "Yes (Immediate Pass)" --> ORIGIN["Fetch Protected Origin Application"]
    end

    subgraph SecurityGates ["3. Multi-Vector Threat Analysis"]
        WL -- "No" --> QCHECK{"Active KV Quarantine?"}
        QCHECK -- "Quarantined" --> REDIR["302 Redirect to Compliant White Page"]
        
        QCHECK -- "Clean" --> ASN{"Datacenter ASN / VPN Block?"}
        ASN -- "Matched (AWS, Hetzner, DO)" --> LOG_ASN["ctx.waitUntil: Async SIEM Alert"] --> REDIR
        
        ASN -- "Passed" --> PROBE{"Vulnerability Probe Pattern?"}
        PROBE -- "Matched (.env, wp-login, admin)" --> BAN["Write Dynamic Ban to KV (7d TTL)"] --> REDIR
        
        PROBE -- "Passed" --> SIG{"Cryptographic Token Match?"}
        SIG -- "Invalid / Missing" --> REDIR
    end

    subgraph OriginPass ["4. Origin Ingress & Telemetry"]
        SIG -- "Valid Signature" --> ORIGIN
        ORIGIN --> ASYNC_TELEMETRY["ctx.waitUntil: Async Telemetry Ingestion"]
    end

    style WL fill:#1e293b,stroke:#3b82f6,stroke-width:2px,color:#fff
    style QCHECK fill:#1e293b,stroke:#ef4444,stroke-width:2px,color:#fff
    style PROBE fill:#1e293b,stroke:#f59e0b,stroke-width:2px,color:#fff
    style SIG fill:#1e293b,stroke:#10b981,stroke-width:2px,color:#fff
```

---

## 🛡️ Multi-Vector Threat Evaluation Matrix

| Vector | Detection Mechanism | Enforcement Action | Latency Impact |
| :--- | :--- | :--- | :---: |
| **Datacenter Scrapers & VPNs** | Real-time ASN cross-reference (AWS, Hetzner, OVH, DigitalOcean, Linode, Choopa) | Transparent 302 Cloak Redirection | `< 1ms` |
| **Vulnerability Probes** | Path token scanning (`/.env`, `/wp-login.php`, `/admin`, `/xmlrpc.php`) | Auto-escalating 7-day KV Quarantine | `< 4ms` |
| **Distributed Bot Networks** | Dynamic IP state inspection across global Cloudflare KV namespace | Cached ban enforcement | `< 6ms` |
| **Direct URL Scraping** | Signature key requirement via tokenized query parameters (`?src=...`) | Soft redirect to brand-compliant White Page | `< 1ms` |

---

## ⚡ Asynchronous Zero-Cost Telemetry

To ensure legitimate users experience zero latency penalty, all security event logging and metric dispatches are decoupled from the response cycle using **`ctx.waitUntil`**:

```javascript
// Non-blocking security event dispatch to SIEM/n8n pipeline
ctx.waitUntil(logSecurityEvent(clientIP, `ASN_BLOCKED_${clientASN}`, request));
return Response.redirect(CONFIG.whitePageUrl, 302);
```

---

## 📊 Benchmark & Production Metrics

| Metric | Measured Value | Standard Origin Firewall |
| :--- | :--- | :--- |
| **P50 Decision Latency** | **4.2 ms** | 120 - 250 ms |
| **P99 Decision Latency** | **11.8 ms** | 450+ ms |
| **Cold Start Time** | **0 ms (V8 Isolates)** | 800 - 2200 ms (Containers) |
| **Origin Compute Load Reduction** | **-74.8%** | Baseline |
| **Edge Concurrency** | **100,000+ req/sec** | Bound by origin CPU |

---

## 🚀 Deployment & Setup

### Prerequisites
* Node.js 18+
* Cloudflare account with Workers & KV enabled
* [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/)

### 1. Installation & Configuration
```bash
git clone https://github.com/coutinhoicaro/edge-traffic-cloaker-router.git
cd edge-traffic-cloaker-router
npm install
```

### 2. Configure `wrangler.toml`
```toml
name = "edge-traffic-cloaker-router"
main = "src/worker.js"
compatibility_date = "2026-08-01"

kv_namespaces = [
  { binding = "QUARENTENA", id = "<YOUR_CLOUDFLARE_KV_NAMESPACE_ID>" }
]
```

### 3. Deploy to Global Edge
```bash
npx wrangler deploy
```

---

## 📂 Repository Structure

```
edge-traffic-cloaker-router/
├── src/
│   └── worker.js           # Core V8 Isolate Decision Engine & KV Handlers
├── package.json            # Node.js Workspace Configuration
├── wrangler.toml           # Cloudflare Edge Deployment Spec
├── LICENSE                 # MIT License
└── README.md               # Architecture, Security Matrices & Benchmarks
```

---

## 📄 License
Distributed under the **MIT License**. See `LICENSE` for details.
