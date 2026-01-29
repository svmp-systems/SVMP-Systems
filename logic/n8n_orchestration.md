# SVMP v4.0 | n8n Orchestration Logic

The system is decoupled into three distinct n8n workflows to ensure high-availability and prevent race conditions.

## Workflow A: The Ingestor (Atomic Write)
- **Role:** High-frequency ingestion.
- **Logic:** Performs an atomic `$push` for messages and resets the sliding debounce window.
- **Debounce Formula:** `debounceExpiresAt = Current_Time + 15s`.
- **Constraint:** This workflow never triggers an LLM; it only maintains state.

## Workflow B: The Processor (Deterministic Gate)
- **Role:** Intent matching and AI synthesis.
- **Polling:** Runs on a 20s cron cycle.
- **Mutex Lock:** Immediately sets `processing: true` upon session pickup to prevent duplicate executions.
- **Governance Gate:** Converts the aggregated session history into a vector and checks against the ≥0.75 threshold.

## Workflow C: The Janitor (Maintenance)
- **Role:** Cleanup and Governance Logging.
- **Interval:** Nightly (12:00 AM).
- **Archive:** Moves sessions with 24h of inactivity to `governance_logs` for compliance auditing.
