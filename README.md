# SVMP Systems v4.1
### State-Aware, Multi-Tenant Governance Engine for LLM-Based Systems

> **Status:** Production Grade (Released Feb 2026)  
> **Core Philosophy:** *Transitioning from Semantic Gating to State-Aware, Multi-Tenant Governance.*

SVMP Systems is a reference implementation of a state-aware orchestration layer designed to address the "Reliability Gap" in LLM deployments. Rather than focusing on prompt optimization, the system addresses failure at the structural level.

---

## 01. Executive Summary
For the past four development cycles, SVMP has worked on solving the reliability gap in probabilistic AI.
**The Foundational Pillars (v4.1):**

1.  **The Identity Tuple:** A 3-dimensional coordinate system (`tenantId`, `clientId`, `userId`) that guarantees 100% data privacy between organizations.
2.  **Dynamic Intent Aggregation:** A **Soft Debounce Queue (2.5s)** that merges fragmented user inputs ("multi-bursts") into a single Complete Thought Unit, reducing API overhead by 60%.
3.  **Multi-Cluster Domain Isolation:** Data is now siloed not just by Tenant, but by **Domain** (e.g., `ecom_001` vs `support_002`), ensuring a sales query never retrieves technical support documents.
4.  **Intent Bifurcation:** A "Logic Fork" that structurally separates **Informational** queries (RAG) from **Transactional** intents (Orders). Transactional queries bypass the LLM entirely to eliminate hallucination.

---

## 02. System Visualization (The Logic Fork)

```mermaid
graph TD
    User([User Message]) -->|Ingest (2.5s Debounce)| Ingestor[Workflow A: Ingestor]
    Ingestor -->|Atomic Write| DB[(Session State)]
    DB -->|Trigger| Processor[Workflow B: Processor]
    
    subgraph "The Logic Fork (v4.1)"
    Processor -->|Check Identity & Domain| Router{Intent Bifurcation}
    
    %% Path A: Transactional
    Router -->|Order ID / Refund| PathA[Path A: Transactional]
    PathA -->|API Call| ERP[Client ERP / Shopify]
    ERP -->|Deterministic Resp| UI([User UI])
    
    %% Path B: Informational
    Router -->|General Query| PathB[Path B: Informational]
    PathB -->|Vector Search| RAG[RAG Pipeline]
    RAG -->|Similarity Check| Gate{Score >= 0.75?}
    
    Gate -- Yes --> LLM[LLM Synthesis]
    Gate -- No --> Escalation[Human Agent]
    end
    
    LLM --> UI
    Escalation --> UI

```

---

## 03. The Tri-Workflow Engine

SVMP decouples concerns into three independent workflows to ensure scalability.

### Workflow A: The Ingestor (Throughput)

* **Role:** Anchors incoming events into a governed session state.
* **Mechanism:** Applies the **2.5s Soft Debounce** to capture "Complete Thought Units."
* **Safety:** Every write is strictly filtered by the **Identity Tuple**.

### Workflow B: The Processor (Reasoning)

* **Role:** Intent classification and logic execution.
* **Mechanism:** Triggered by a 1s Cron. Acquires an **Atomic MUTEX Lock** (`processing: true`) to guarantee **Exactly-Once Processing**.
* **Logic Fork:** Splits traffic into **Path A** (API) or **Path B** (RAG >= 0.75).

### Workflow C: The Janitor (Lifecycle)

* **Role:** Data hygiene and compliance.
* **Mechanism:** A 24-hour cycle that purges stale sessions from Hot Storage and archives them to the Immutable Governance Ledger.

---

## 04. Repository Structure

This repository serves as the technical specification for the v4.1 architecture.

* `ARCHITECTURE.md` — Deep dive into invariants and concurrency models.
* `logic/` — Decision engines (`n8n_orchestration`) and bifurcation logic.
* `workflows/` — Maintenance cycles (The Janitor).
* `schemas/` — JSON definitions for Sessions, Tenants, and Knowledge Base.

---

## 05. Evolutionary Roadmap

### v4.1 (Current Production)

* **New:** Intent Bifurcation (Conditional Execution Pipeline).
* **New:** Multi-Cluster Domain Isolation (`domainId` routing).
* **Focus:** Removing "Transactional Hallucination" via API bypass.

### v4.0 (Conceptual Foundation)

* **Introduced:** Soft Debounce (2.5s) & Mutex Locking.
* **Introduced:** Strict Multi-Tenant Isolation.

### v3.0 (Legacy Reference)

* **Introduced:** The 0.75 Similarity Gate.
* **Status:** Validated baseline; retired for v4.1.

---

## Authors

**Pranav H**
Lead Architect

**Samrth D Magi**
Product Lead & systems architect 

**Shravan Kumar**
Operations Lead
