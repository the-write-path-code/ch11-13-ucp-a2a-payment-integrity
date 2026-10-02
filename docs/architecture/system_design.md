# UCP Agent Architecture & Integrity Design

This document visualizes the architectural patterns used to guarantee payment integrity and state consistency in the Universal Commerce Protocol (UCP) Agent.

## 1. Deployment Topology (Multi-Worker)

Illustrates why in-memory locks fail and why we enforce safety at the Persistence Layer.

```mermaid
flowchart TD
    LB["<div style='min-width: 320px;'><b>Client Ingress & Load Balancer (Port 8000)</b><br/>Distributes concurrent requests & retries across workers</div>"]
    
    subgraph Workers ["Application Layer: Stateless Workers (No Shared Memory)"]
        direction LR
        W1["<div style='min-width: 170px;'><b>Worker 1</b><br/>Stale Mutation</div>"]
        W2["<div style='min-width: 170px;'><b>Worker 2</b><br/>Winning Mutation</div>"]
        W3["<div style='min-width: 170px;'><b>Worker 3</b><br/>Initial Request</div>"]
        W4["<div style='min-width: 170px;'><b>Worker 4</b><br/>Duplicate Retry</div>"]
    end
    
    LB --> W1
    LB --> W2
    LB --> W3
    LB --> W4
    
    DB[("<div style='min-width: 440px;'><b>Persistence Layer: SQLite Database (Source of Truth)</b><br/>• UNIQUE Index on checkout_id<br/>• Version Column for Optimistic Concurrency</div>")]
    
    W1 -.->|"<b>StateConflict</b><br/>(Ver Mismatch)"| DB
    W2 -->|"<b>Update (Ver=2)</b><br/>Commit OK"| DB
    W3 -->|"<b>Insert (Ver=1)</b><br/>201 Created"| DB
    W4 -.->|"<b>IntegrityError</b><br/>(Duplicate)"| DB

    classDef client fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef worker fill:#EBF5FF,stroke:#2563EB,color:#000000,stroke-width:1.5px
    classDef winner fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px
    classDef db fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px

    class LB client
    class W1,W4 worker
    class W2,W3 winner
    class DB db
```

---

## 2. Retry Storm Sequence (Payment Integrity)

Shows how the Database Unique Constraint acts as the "Atomic Guard" against duplicate payments (Double Spend).

```mermaid
sequenceDiagram
    autonumber
    actor Client as Client Agent
    participant W1 as Worker 1
    participant W2 as Worker 2
    participant DB as SQLite DB

    note over Client, DB: Scenario: Retry Storm (Duplicate Delivery of msg_101)

    Client->>W1: Initial Request (msg_101)
    Client->>W2: Retry after timeout (msg_101)

    note over W1, DB: Workers 1 and 2 process checkout concurrently

    W1->>DB: INSERT order (checkout_123)
    W2->>DB: INSERT order (checkout_123)

    note over W2, DB: Constraint Gate: UNIQUE(checkout_id)

    DB-->>W1: 201 Created (Order 1001)
    DB--xW2: UNIQUE Constraint Failed

    W1-->>Client: Return Order 1001 (New Order)

    note over W2, DB: Catch DuplicateOrderError and fetch existing order
    W2->>DB: SELECT order WHERE checkout_id=123
    DB-->>W2: Return Order 1001
    W2-->>Client: Return Order 1001 (Idempotent Receipt)

    note over Client, DB: Outcome: Exactly-once commitment (1 order, 2 identical receipts)
```

---

## 3. Mutation Race Sequence (Optimistic Concurrency)

Shows how Versioning (OCC) detects dirty reads when an "Add Item" request interleaves with a "Payment" request.

```mermaid
sequenceDiagram
    autonumber
    participant Buyer as Buyer Agent
    participant Svc as Checkout Service
    participant Other as Concurrent Mutator
    participant DB as SQLite DB

    note over Buyer, DB: Scenario: Interleaving Cart Mutation During Payment

    Buyer->>Svc: Read Cart to start payment
    Svc->>DB: SELECT cart (version 1, USD 100)
    DB-->>Svc: Cart State (version 1, USD 100)
    Svc-->>Buyer: Cart Snapshot (Expected version 1)

    note over Buyer, DB: Race Window: Background Mutation
    Other->>Svc: Add emergency tubing (+50 USD)
    Svc->>DB: UPDATE cart (version 2, USD 150)
    DB-->>Svc: Commit OK (version 2)

    note over Buyer, DB: Commit Gate: Validate Freshness
    Buyer->>Svc: Commit Order (Assert version 1, USD 100)
    Svc->>DB: Validate: Stored v2 == Expected v1?
    DB--xSvc: Conflict: Version Mismatch (2 != 1)

    note over Svc, DB: Catch StateConflictError (Stale read rejected)
    Svc-->>Buyer: 409 Conflict (Cart Modified, Please Retry)
```

---

## 4. Payment Integrity Guard Logic (Flowchart)

The decision tree for handling incoming requests safely.

```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
flowchart TD
    Start([User Agent Sends 'Complete Checkout']) --> CheckIdem{"Check In-Memory<br/>Idempotency Key?"}
    
    CheckIdem -- "Not Found" --> AttemptCreate["Attempt Atomic Insert<br/>WHERE version = expected"]
    
    AttemptCreate --> CheckResult{"DB Result?"}
    
    CheckResult -- "Success" --> UpdateStatus["Update Checkout Status"]
    UpdateStatus --> StoreResponse["Store in Cache"]
    StoreResponse --> ReturnSuccess([Return Success])
    
    %% The New Path for Mutation Race
    CheckResult -- "Version Mismatch" --> Conflict["Return 409 Conflict<br/>(State Changed)"]
    Conflict --> ReturnError([Client Must Retry])
    
    CheckResult -- "Unique Constraint" --> CatchError["Catch DuplicateOrderError"]
    CatchError --> FetchExisting["Fetch Existing Order"]
    FetchExisting --> ReturnExisting["Return Existing Order"]
    
    ReturnExisting --> ReturnSuccess
    CheckIdem -- "Found" --> ReturnCached["Return Cached"]
    ReturnCached --> ReturnSuccess
```

---

## 5. State Diagram (Order Lifecycle)

Formalizing the idempotent state transition: "Failure to Create" is a valid path to "Success".

```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
stateDiagram
  direction TB
  state FetchExisting {
    direction TB
    [*] --> LookupOrder
    LookupOrder --> ReturnOrder
[*]    LookupOrder
    ReturnOrder
  }
  [*] --> CheckoutPending
  CheckoutPending --> Race:Complete Checkout
  Race --> OrderCreated:Insert Success (Winner)
  Race --> FetchExisting:Insert Failed (Unique Violation)
  FetchExisting --> OrderCreated:Return Existing Order
  OrderCreated --> [*]
  note right of Race 
  Guarded by DB Unique Constraint
        on checkout_id
  end note
```

---

## 6. Conflict Recovery After OCC Rejection

Shows the post-conflict path after a stale write is rejected: the payment attempt arrives with an expected checkout version, the persistence layer detects that storage has advanced, the write fails with a conflict, and the caller must refetch current state before retrying or asking for reconfirmation.

```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
sequenceDiagram
    participant Agent as Agent / Client
    participant API as Checkout Service
    participant DB as Persistence Layer
    participant User as User / Upstream Workflow

    Agent->>API: Complete checkout(checkout_id, expected_version = n)
    API->>DB: create_order_safe(checkout_id, expected_version = n)
    DB->>DB: Re-read checkout version and total

    alt State still matches version n
        DB-->>API: Commit order
        API-->>Agent: Success, return committed order
    else State changed to version n+1
        DB-->>API: 409 StateConflictError
        API-->>Agent: Conflict, latest version = n+1
        Agent->>DB: Refetch latest checkout state

        alt Updated state still acceptable
            Agent->>API: Retry complete checkout(expected_version = n+1)
        else Approval required
            Agent->>User: Ask for reconfirmation
        end
    end

```

---

## 7. Retry-safe checkout completion across concurrent workers.
Stable request identity distinguishes repeated delivery from new intent, a durable idempotency ledger records the winning business action, and the database uniqueness rule on orders(checkout_id) ensures that concurrent writers converge on one committed order.

```mermaid
sequenceDiagram
    autonumber
    participant C as Client Agent
    participant W1 as Worker A
    participant W2 as Worker B
    participant L as Idempotency Ledger
    participant DB as SQLite DB (UNIQUE)

    C->>W1: complete_checkout(msg_101)
    W1->>L: lookup IdempotencyKey
    L-->>W1: no committed record
    W1->>DB: INSERT order (checkout_123)

    note over C: acknowledgement delayed or lost

    C->>W2: retry checkout(msg_101)
    W2->>L: lookup same key
    L-->>W2: race window still open
    W2->>DB: INSERT order (checkout_123)

    DB-->>W1: insert succeeds
    W1->>L: commit key -> Order 1001

    DB--xW2: UNIQUE constraint violation
    W2->>DB: SELECT WHERE checkout_id=123
    DB-->>W2: return Order 1001
    W2->>L: reconcile key -> Order 1001

    W1-->>C: Order 1001 (Winner)
    W2-->>C: Order 1001 (Idempotent Receipt)
```

---

## 8. Fast path versus atomic slow path. 
Early idempotency lookup and in-memory locking can intercept likely duplicates, but only the database unique constraint on orders(checkout_id) settles the race across workers. The winning request commits the order, and the losing request re-reads that canonical row and returns the same business result. 

```mermaid
flowchart TD
    Req["<div style='min-width: 380px;'><b>Checkout Request & Idempotency Inspection</b><br/>POST complete_checkout(id) triggers in-memory key lookup</div>"]

    Fast["<div style='min-width: 260px;'><b>Fast Path (Replay Cache Hit)</b><br/>Return recorded response immediately</div>"]
    Slow["<div style='min-width: 280px;'><b>Slow Path (Atomic Coordination)</b><br/>Acquire advisory lock & attempt INSERT</div>"]

    Req -->|"Hit (Replay)"| Fast
    Req -->|"Miss (New / Concurrent)"| Slow

    DB["<div style='min-width: 440px;'><b>Database Uniqueness Boundary: UNIQUE(checkout_id)</b><br/>Shared SQLite engine settles race across all concurrent workers</div>"]
    Slow --> DB

    subgraph WinnerPath ["Winning Execution Path"]
        direction TB
        W1["<div style='min-width: 240px;'><b>Insert Succeeded (Order Committed)</b><br/>Order row written & replay record cached</div>"]
        W2["<div style='min-width: 240px;'><b>Return Completed Checkout</b><br/>201 Created response sent to caller</div>"]
        W1 --> W2
    end

    subgraph LoserPath ["Concurrent Collision Path"]
        direction TB
        L1["<div style='min-width: 240px;'><b>DuplicateOrderError Caught</b><br/>UNIQUE constraint violation on checkout_id</div>"]
        L2["<div style='min-width: 240px;'><b>Return Canonical Order</b><br/>SELECT by checkout_id -> 200 OK receipt</div>"]
        L1 --> L2
    end

    DB -->|"Winner (First Commit)"| W1
    DB -->|"Loser (Duplicate Insert)"| L1

    classDef req fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef coord fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px
    classDef db fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px
    classDef winner fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px
    classDef loser fill:#FEE2E2,stroke:#DC2626,color:#000000,stroke-width:1.5px

    class Req req
    class Fast,W1,W2 winner
    class Slow coord
    class DB db
    class L1 loser
    class L2 coord
```

---

## 9. The schema settles the race. 
Advisory replay checks may detect likely duplicates early, but the unique index on orders(checkout_id) is the shared enforcement point that allows one committed order row and forces the loser to reconcile by re-reading the canonical order.

```mermaid
flowchart TD
    subgraph Ingress ["Concurrent Worker Ingress (Stateless Tier)"]
        direction LR
        A["<div style='min-width: 320px;'><b>Worker A: Initial Request</b><br/>Binds request identity & attempts store insert</div>"]
        B["<div style='min-width: 320px;'><b>Worker B: Concurrent Retry</b><br/>Same request identity & attempts store insert</div>"]
    end

    DB[("<div style='min-width: 580px;'><b>Database Uniqueness Boundary: UNIQUE(checkout_id)</b><br/>Shared SQLite engine settles concurrency race at transaction boundary</div>")]

    A --> DB
    B --> DB

    subgraph Resolution ["Atomic Storage Resolution"]
        direction LR
        W["<div style='min-width: 320px;'><b>One Order Row Commits</b><br/>Winner writes canonical row into orders table</div>"]
        L["<div style='min-width: 320px;'><b>DuplicateOrderError Caught</b><br/>IntegrityError caught; calls get_order_by_checkout_id</div>"]
    end

    DB -->|"Winner (First Commit)"| W
    DB -->|"Loser (Duplicate Insert)"| L

    Final["<div style='min-width: 580px;'><b>Converged Result: Canonical Order Exists</b><br/>Repeated deliveries converge on exactly one durable business outcome</div>"]

    W --> Final
    L --> Final

    classDef worker fill:#EBF5FF,stroke:#2563EB,color:#000000,stroke-width:1.5px
    classDef db fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:1.5px
    classDef win fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px
    classDef lose fill:#FEE2E2,stroke:#DC2626,color:#000000,stroke-width:1.5px
    classDef final fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px

    class A,B worker
    class DB db
    class W win
    class L lose
    class Final final
```

---

## 10. Commit authority stays below the model boundary. 
The model may select complete_checkout, and local replay checks may reduce duplicate work, but only the database commit boundary can decide whether a new order is created or a duplicate must reconcile to the canonical row.

```mermaid
flowchart TD
    Ingress["<div style='min-width: 680px;'><b>1. Model Action Selection & Service Identity Binding</b><br/>User triggers intent & prompt -> LLM selects complete_checkout (advisory)<br/>Service seam binds durable request identity (context_id, message_id)</div>"]

    AdvReplay["<div style='min-width: 320px;'><b>Replay Lookup (Advisory)</b><br/>Idempotency ledger early check for recorded result</div>"]
    AdvLock["<div style='min-width: 320px;'><b>InMemoryLockManager (Advisory)</b><br/>Local worker process lock to reduce concurrency churn</div>"]

    Ingress --> AdvReplay
    Ingress --> AdvLock

    StoreInsert["<div style='min-width: 680px;'><b>Store Issues Order INSERT</b><br/>Translates business action into database write request</div>"]

    AdvReplay --> StoreInsert
    AdvLock --> StoreInsert

    DB[("<div style='min-width: 680px;'><b>2. Database Commit Boundary: orders table UNIQUE(checkout_id)</b><br/>Commit authority stays strictly below the model boundary in durable storage engine</div>")]

    StoreInsert --> DB

    Win["<div style='min-width: 320px;'><b>Commit Succeeds (Winner)</b><br/>First write succeeds; order row committed durably</div>"]
    Lose["<div style='min-width: 320px;'><b>Duplicate Rejected (Loser)</b><br/>DuplicateOrderError caught; re-reads get_order_by_checkout_id</div>"]

    DB -->|"Commit succeeds"| Win
    DB -->|"Duplicate rejected"| Lose

    Final["<div style='min-width: 680px;'><b>3. Return Canonical Durable Result & Post-Commit Explanation</b><br/>Both paths converge on exact same committed order; LLM explains result post-commit</div>"]

    Win --> Final
    Lose --> Final

    classDef ingress fill:#F3F4F6,stroke:#4B5563,color:#000000,stroke-width:1.5px
    classDef advisory fill:#FEF3C7,stroke:#D97706,color:#000000,stroke-width:1.5px
    classDef store fill:#E0F2FE,stroke:#0284C7,color:#000000,stroke-width:1.5px
    classDef db fill:#FEF9C3,stroke:#CA8A04,color:#000000,stroke-width:2px
    classDef win fill:#DCFCE7,stroke:#15803D,color:#000000,stroke-width:1.5px
    classDef lose fill:#FEE2E2,stroke:#DC2626,color:#000000,stroke-width:1.5px
    classDef final fill:#EDE9FE,stroke:#7C3AED,color:#000000,stroke-width:1.5px

    class Ingress ingress
    class AdvReplay,AdvLock advisory
    class StoreInsert store
    class DB db
    class Win win
    class Lose lose
    class Final final
```

---

## 11. Checkout version lifecycle. 
A payment attempt begins from one persisted checkout version. If another actor mutates the checkout before commit, the version advances and the original payment attempt becomes stale. The order path must compare the caller's last-seen version with storage before creating the order.

```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
stateDiagram-v2
    [*] --> Incomplete_v1: create_checkout()

    Incomplete_v1 --> Incomplete_v2: add_to_checkout(), save_checkout()
    Incomplete_v2 --> Ready_v3: start_payment(), save_checkout()

    Ready_v3 --> Mutated_v4: concurrent cart change, save_checkout(), version = 4
    Ready_v3 --> CommitCheck: complete_checkout(), expected_version = 3

    CommitCheck --> Completed_v4: stored version = 3, order created, save_checkout()
    CommitCheck --> Conflict: stored version != 3, raise StateConflictError

    note right of Ready_v3
      Payment starts from a specific
      persisted checkout version.
    end note

    note right of Mutated_v4
      Another actor changes the checkout
      before payment commits.
    end note

    note right of Conflict
      The original write is stale and must
      refresh state before any next step.
    end note

```

##12. Persistence-boundary validation in the repository
```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
sequenceDiagram
    participant S as CheckoutService
    participant Store as SQLiteStore
    participant DB as SQLite DB

    S->>Store: create_order_safe(checkout, expected_version)
    Store->>DB: SELECT version, total_cents FROM checkouts WHERE checkout_id=?
    DB-->>Store: real_version, real_total

    alt real_version != expected_version or real_total != checkout.total_cents
        Store-->>S: raise StateConflictError
        Note over S,DB: No order insert. Stale write rejected.
    else state matches
        Note over Store,DB: Validation connection closes here
        Store->>DB: INSERT INTO orders (order_id, checkout_id, total_cents, permalink_url)
        DB-->>Store: commit
        Store-->>S: return Order
    end

    Note over Store,DB: Gap between validation and insert is the current commit boundary
```

##13. Conflict recovery in complete_checkout()
```mermaid
%%{init: {"theme": "neutral", "themeVariables": {"fontFamily": "Arial, Helvetica, sans-serif", "fontSize": "11px", "actorFontSize": "11px", "noteFontSize": "10px", "messageFontSize": "10px"}}}%%
flowchart TD
    A["create_order_safe() raises StateConflictError"] --> B["complete_checkout() catches the exception"]
    B --> C["get_checkout(checkout_id) re-reads the persisted row"]
    C --> D["return the current Checkout to the caller"]
    A -.-> E["no order inserted"]
    D -.-> F["caller policy: retry from fresh state, escalate, or abandon"]
```
