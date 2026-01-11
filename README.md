```mermaid
graph TD
    User([User Query]) -->|HTTP POST| Orch{Go Orchestrator}

    %% Tier 1: Cache
    Orch -->|Check Cache| Cache[(Redis Cache)]
    Cache -->|Hit (<1ms)| Orch
    Orch -->|Return Verdict| User

    %% Tier 2: Scatter-Gather
    Cache -->|Miss| DB[(PostgreSQL)]
    Cache -->|Miss| BERT[Python BERT Service]

    BERT -->|Analyze| Decision{Confidence > 80%?}

    %% Fast Path
    Decision -->|Yes: Cache & Return| Orch

    %% Slow Path (Async)
    Decision -->|No: Enqueue| Queue[[Redis Queue]]
    Orch -->|Return Pending| User

    %% Tier 3: Deep Analysis
    Queue -->|Consume| Worker[Deep Blue Worker]
    Worker -->|Search & Reason| LLM[Gemini + DuckDuckGo]
    LLM -->|Write-Through| Cache
    LLM -->|Audit Log| DB

```
