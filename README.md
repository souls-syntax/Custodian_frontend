```mermaid
graph TD
    User([User Query]) -->|HTTP POST| Orch{Go Orchestrator}
    
    %% Tier 1: Cache
    Orch -->|1. Check| Cache[(Redis Cache)]
    Cache -- Hit -->|Return < 1ms| Orch
    Orch -- Hit --> User

    %% Tier 2: Scatter Gather
    Cache -- Miss -->|2. Dispatch| DB[(PostgreSQL)]
    Cache -- Miss -->|2. Dispatch| BERT[Python BERT Service]
    
    BERT -->|3. Analysis| Decision{Confidence > 80%?}
    
    %% Fast Path
    Decision -- Yes -->|4. Return & Cache| Orch
    
    %% Slow Path (Async)
    Decision -- No -->|5. Enqueue| Queue[[Redis Queue]]
    Orch -->|Return 'Processing'| User
    
    %% Tier 3: Deep Blue
    Queue -->|6. Consume| Worker[Deep Blue Worker]
    Worker -->|7. Search & Reason| LLM[Gemini + DuckDuckGo]
    LLM -->|8. Write-Through| Cache
    LLM -->|8. Update Audit| DB
```
