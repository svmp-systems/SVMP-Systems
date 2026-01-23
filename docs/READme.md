# Contents

## SVMP v3.0 Whitepaper

**Architecture Model:** Message-centric

**Characteristics:**
Best-effort orchestration with explicit governance guardrails
LLM similarity outputs directly acted upon for routing and intent matching
Governance gates existed, but control logic was tightly coupled to message flow

**Limitations:**
Message-level processing made it difficult to reason about multi-step interactions
Scaling introduced auditability, reliability, and cross-tenant risk
Edge-case handling required ad-hoc workflow branching

**Status:** Historical reference
Retained to document architectural learning and the rationale for the v4 rewrite

## SVMP v4.0 Whitepaper (Upcoming)

Architecture Model: Session-centric (unit of truth = session)

**Core Principles:**
Deterministic orchestration enforced at the workflow layer
Strict multi-tenant isolation across storage and logic
Soft-debounce batching to aggregate rapid user inputs
Explicit human-escalation outcomes for ambiguous or low-confidence cases

**Focus:**
Architectural decisions, governance philosophy, and failure modes
Exactly-once processing guarantees at the orchestration level

**Notes:**
Core execution and deployment logic intentionally withheld due to potential productization

## SVMP v4.1 Notes (In Progress)

**Scope:**
Incorporation of e-commerce and D2C-specific edge cases (e.g., order-ID-dependent flows)
Conditional enrichment logic layered onto the existing v4.0 workflow

**Design Constraint:**
Maintain a single common workflow across all tenants
Avoid workflow fragmentation for minor domain-specific variations

**Trigger:**
Expansion of SVMP’s market scope to include e-commerce / D2C automation

**What Does NOT Change:**
Session-centric architecture
Governance philosophy
Multi-tenant isolation guarantees
Deterministic orchestration model

**Purpose:**
Preserve architectural generality while supporting domain-specific requirements
Documentation Philosophy

## SVMP documentation prioritizes:
Determinism over black-box behavior
Auditability over implicit logic
Explicit tradeoff documentation
Versioned architectural reasoning
This repository intentionally publishes architecture, governance logic, and iteration rationale, while keeping core execution and deployment logic private.

## Version Timeline (High Level)
v3.0 — Message-centric, best-effort orchestration with guardrails
v4.0 — Session-centric deterministic orchestration and governance
v4.1 — Domain-specific edge-case integration without workflow fragmentation

## Notes on Scope
SVMP is an evolving system used across:
Internal infrastructure experiments
NGO operational workflows
Early-stage labor marketplace and e-commerce automation
This documentation exists to provide transparent evidence of systems thinking and architectural iteration, not as a turnkey open-source product.
