# 🛡️ SVMP v4.1: Architecture & Governance Engine

> **System Status:** Production Grade  
> **Core Philosophy:** *Large Language Models (LLMs) are non-deterministic components and must be managed as untrusted workers inside a deterministic system.*

SVMP (Semantic Vector Mapping Protocol) is a **governance layer** designed to bridge the "Reliability Gap" in enterprise AI deployments by constraining LLM behavior through structural architecture rather than prompt engineering.

---

## 📑 Table of Contents
- [1. Design Premise](#1-design-premise)
- [2. The Tri-Workflow Engine](#2-the-tri-workflow-engine)
- [3. Architectural Invariants](#3-architectural-invariants)
- [4. The Identity Tuple (Hard Siloing)](#4-the-identity-tuple-hard-siloing)
- [5. Soft Debounce & Aggregation](#5-soft-debounce--aggregation)
- [6. Intent Bifurcation & Governance](#6-intent-bifurcation--governance)
- [7. Forensic Auditability](#7-forensic-auditability)
- [8. Roadmap & Scalability](#8-roadmap--scalability)

---

## 1. Design Premise
Most production LLM failures stem from systemic issues, not model weakness:
* **Race Conditions:** Responding multiple times to fragmented user bursts.
* **Context Leakage:** Data bleeding across tenants or domains.
* **Transactional Hallucination:** Inventing facts instead of querying a Source of Truth (SoT).

SVMP solves this by treating the LLM as a "service" that can only be reached after passing through deterministic gates.

---

## 2. The Tri-Workflow Engine
SVMP decouples concerns into three independent workflows that scale horizontally.

### A. Workflow: The Ingestor (Throughput)
* **Goal:** High-concurrency message acceptance.
* **Action:** Accepts webhooks, computes the **Identity Tuple**, and performs an atomic upsert into `session_state`.
* **Key Choice:** Never blocks on LLM execution. Ingestion scales independently of model latency.

### B. Workflow: The Processor (Correctness)
* **Goal:** Exactly-once execution.
* **Action:** Triggered by a 1s cron/event-bus. Acquires a **MUTEX lock** (`processing: false → true`), aggregates messages, and routes via the Intent Engine.
* **Key Choice:** Lock-based state transitions prevent double-processing during high-load bursts.

### C. Workflow: The Janitor (Hygiene)
* **Goal:** Compliance and performance.
* **Action:** 24h cycle to purge stale session states. Ensures final states are mirrored to immutable governance logs.

---

## 3. Architectural Invariants
The system enforces these five non-negotiable rules structurally:

| Invariant | Implementation |
| :--- | :--- |
| **Identity Mandatory** | Every operation is scoped by `(tenantId, clientId, userId)`. |
| **State Precedes Gen** | No LLM call occurs without a persisted session state. |
| **Exactly-Once** | Atomic state transitions prevent double-processing. |
| **Transactional Truth** | Real-world data (Orders/CRM) must be fetched via API, not LLM memory. |
| **Default to Silence** | Similarity < 0.75 triggers immediate human escalation. |

---

## 4. The Identity Tuple (Hard Siloing)
The Identity Tuple `(tenantId, clientId, userId)` is the **primary key** of the system. 

Unlike prompt-based isolation—which is vulnerable to injection—SVMP enforces isolation at the **database query layer**. Cross-tenant data leakage is structurally impossible because the application cannot "see" data outside of its specific tuple.

---

## 5. Soft Debounce & Aggregation
Users rarely send complete thoughts in one message (e.g., *"Hi"* → *"I need help"* → *"order 123"*).

**The Solution:** A sliding **2.5s Soft Debounce Window**.
1. Every inbound message appends to a buffer and resets `debounceExpiresAt = now + 2.5s`.
2. The **Processor** only triggers once the window expires.
3. **Impact:** 40–60% reduction in token costs and near-zero "hallucination-by-fragmentation."

---

## 6. Intent Bifurcation & Governance
SVMP explicitly separates **Informational** and **Transactional** paths before any text generation occurs.

### 🛣️ Transactional Path (Deterministic)
If the Intent Classifier detects a need for an Order/User ID:
* **Check:** Is the ID present in the `combinedText`?
* **Action:** Fetch live data from Shopify/CRM API.
* **Safety:** If ID is missing, the system asks for it and stops—no guessing.

### 🧠 Informational Path (Probabilistic)
Queries run through the **Vector Similarity Gate**:
* **Cosine Similarity ≥ 0.75:** System auto-replies using verified Knowledge Base clusters.
* **Cosine Similarity < 0.75:** System freezes and escalates to a human via Slack.

---

## 7. Forensic Auditability
Every action is recorded in `governance_logs` as an append-only ledger:
* **Input:** Aggregated raw text.
* **Scoring:** Similarity match & logic branch taken.
* **Attribution:** Document ID or API endpoint used as the source.
* **Identity:** The Tuple responsible for the event.

---

## 8. Roadmap & Scalability
The architecture is designed to evolve without breaking core invariants:
* **Redis Integration:** Transitioning from 1s Cron to Redis `SETEX` for sub-second latency.
* **Horizontal Sharding:** Distributing `session_state` by `tenantId` clusters.
* **Deterministic AI Middleware:** Exposing SVMP as a standardized API for external chatbot UIs.

---
*© 2026 SVMP Systems. Authored by Pranav H.*
