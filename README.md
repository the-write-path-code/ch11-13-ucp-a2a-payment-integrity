# Chapters 11–13: Payment Integrity, Idempotency, and Race Conditions

Companion code for *Building Safe Agentic AI for Enterprise Systems* by Mohit Aggarwal.

This repository demonstrates what changes when an agent can initiate a write rather than only generate text. The running checkout example shows three related but separate problems: repeated delivery of one logical purchase, concurrent attempts to create the same order, and a write made against checkout state that changed after it was read.

The implementation puts the final duplicate-order barrier at the persistence layer. A model may select `complete_checkout`. It does not commit the order. The service establishes request identity, and the database decides whether an order row may exist.

## What You Will Run

| Chapter | Demonstration | What it shows |
| --- | --- | --- |
| 11, Agentic Commerce and the Double Spend Problem | Retry storms and duplicate mutations | Why automatic retries become business failures once a request can create an order, reserve inventory, or move money. |
| 12, Architectural Patterns for Idempotency | Request identity and unique order commitment | Idempotency keys, replay behavior, local coordination, database uniqueness, and loser reconciliation. |
| 13, Handling Race Conditions in Universal Commerce | Stale-write detection | Version-based optimistic concurrency control (OCC) and the need to reject work based on a checkout state that no longer exists. |

The repository uses an intentionally small local checkout system so the failure conditions are visible. It uses SQLite, FastAPI, asynchronous Python, and concurrent request harnesses. It does not contact a payment provider or move real money.

## Production Warning

Do not adapt this repository directly for payment capture, procurement, inventory allocation, access control, or any other consequential write. It is a teaching implementation that isolates specific controls. A real write path also needs authentication, authorization, input validation, tenant boundaries, audit retention, incident handling, secrets management, provider reconciliation, observability, and recovery procedures.

The repository proves two important properties in its demonstrations: a database unique index can stop two committed orders for one checkout, and a version mismatch can identify a stale view of a checkout. It does not yet put the idempotency-record mutation, order insert, version check, and dependent checkout update into one shared database transaction. Chapter 13 names this gap directly. Treat it as the next hardening task, not as a hidden detail.

## Prerequisites

- Git
- [uv](https://docs.astral.sh/uv/)
- Python 3.11
- No API key, database server, container runtime, or cloud account for the local demonstrations

The repository uses SQLite through `aiosqlite`. The local database file is created during a demonstration run.

## Quick Start

### 1. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### 2. Clone and synchronize the repository

```bash
git clone https://github.com/the-write-path-code/ch11-13-ucp-a2a-payment-integrity.git
cd ch11-13-ucp-a2a-payment-integrity
uv sync
```

The repository includes `uv.lock`. Run `uv sync` after pulling changes so the local environment matches the committed dependency set.

### 3. Run the safe demonstration

```bash
uv run python demo.py
```

Start here. It shows a guarded checkout completion path and the expected result when the same logical completion is delivered more than once.

### 4. Run the failure demonstration

```bash
uv run python demo_fail.py
```

This intentionally exercises the unsafe path. It demonstrates why read-then-insert logic, process-local coordination, and retry behavior do not establish system-wide uniqueness.

The failure script can create duplicate orders in its unsafe configuration. It uses a local test database only. Do not point it at a shared, reused, or production database.

## Configuration

The local demonstration uses these settings:

| Variable | Default | Purpose |
| --- | --- | --- |
| `SAFETY_MODE` | `on` | Selects the guarded path. Set to `off` only for the intentional baseline failure demonstration. |
| `DATABASE_URL` | Local SQLite test database | Location of the local test database. |

> **Tip**
>
> Delete the local test database before rerunning an unsafe scenario. A prior order can hide the behavior you are trying to observe.

```bash
rm -f test.db
```

Never set `SAFETY_MODE=off` in a deployed service. It exists only to make the failure path reproducible and testable.

## Run the Chapter Demonstrations

### 1. Retry Storm and Duplicate Orders, Chapter 11

Run the failure demonstration to see the basic hazard:

```bash
uv run python demo_fail.py
```

The harness submits concurrent completion attempts for the same checkout. In unsafe mode, each worker can read the checkout before another worker commits an order. Each then treats its own request as a new action.

The question is not how many HTTP requests arrived. The question is whether those requests represent one purchase intent or several. A retry-safe service identifies business intent and converges repeated delivery onto one durable result.

### 2. Idempotent Completion, Chapter 12

Run the safe demonstration:

```bash
uv run python demo.py
```

The guarded flow has two layers:

1. An idempotency key identifies one logical command, such as `(context_id, message_id, complete_checkout)`.
2. A unique index on `orders(checkout_id)` prevents more than one committed order for the same checkout.

An early idempotency lookup is useful, but it is not the final enforcement point. Two workers can both miss a replay record before either stores it. The database unique constraint is the shared barrier every competing write must cross.

When the second worker loses the uniqueness race, it must not return a vague failure. It reads the committed order and returns the canonical result. This is loser reconciliation: the repeated request converges on the winner's outcome.

### 3. Fast Path and Atomic Slow Path, Chapter 12

The repository distinguishes two paths:

| Path | Responsibility | Limitation |
| --- | --- | --- |
| Fast path | Early lookup of an idempotency record and local in-process locking | Reduces duplicate work, but cannot settle a race across workers or restarts |
| Slow path | Database insert protected by `UNIQUE(orders.checkout_id)` | Shared persistence boundary that determines whether an order may commit |

A process-local `asyncio.Lock` can serialize coroutines inside one worker. It cannot coordinate two Uvicorn workers or two Cloud Run instances because those processes do not share memory.

### 4. Mutation Race and Stale State, Chapter 13

The stale-write scenario is different from duplicate delivery. One worker may have one valid request and still be wrong at commit time because another actor changed the checkout while the first worker was processing it.

The guarded path carries the checkout version it read as `expected_version`. Before creating an order, the store compares that value with the version currently in storage. A mismatch raises `StateConflictError`, and the caller re-reads the current checkout rather than committing against stale state.

The practical rule is:

```text
A unique order is not necessarily a valid order.
```

Idempotency prevents duplicate commitment of one intent. OCC prevents one commitment against an outdated state. The write path needs both.

### 5. Run the Multi-Worker Server, Optional

Start the guarded FastAPI application with multiple workers:

```bash
uv run uvicorn app.main:create_app --factory --host 0.0.0.0 --port 8000 --workers 4
```

Use this only with the local test database and the repository's harnesses. The purpose is to make worker-local state and concurrent persistence behavior visible.

To run the intentionally unsafe baseline, start a separate local test run with `SAFETY_MODE=off`:

```bash
rm -f test.db
SAFETY_MODE=off uv run uvicorn app.main:create_app --factory --host 0.0.0.0 --port 8000 --workers 4
```

Do not run the safe and unsafe versions against the same database at the same time.

### 6. Run the Experiment Sweep, Optional

The repository includes a concurrency experiment runner. Use the package entry point or the experiment module documented in `experiments/README.md`:

```bash
uv run ucp-agent --help
```

The recorded study runs retry-storm, concurrent-race, and mutation-race scenarios across increasing request counts. Review the experiment configuration and outputs before treating any result as a performance claim.

## Expected Results

### Retry storm

Under the unsafe baseline, repeated delivery can produce more than one order for the same checkout. Under the guarded path, the unique index admits one committed order and forces subsequent insert attempts into reconciliation.

The repository's recorded experiment reports 800 unique orders from 800 duplicate completion requests in its unsafe retry-storm scenario, and one unique order under the hardened configuration. These are results of the repository's four-worker test conditions, not a general statement about agentic commerce or databases.

### Concurrent race

Two or more workers can observe no completed order in their local view and race toward insertion. In the guarded path, one insert succeeds. The loser receives a uniqueness conflict, reads the canonical order, and returns that result.

### Mutation race

A checkout can change after one actor reads it and before that actor tries to complete payment. The order attempt should be rejected when its `expected_version` differs from the version currently in storage. The correct next step is to re-read state, re-evaluate the action, and obtain reconfirmation if required.

## Run the Tests

```bash
uv run pytest
```

To include coverage when the project configuration supports it:

```bash
uv run pytest tests -v --cov=app
```

Run tests before changing idempotency keys, order uniqueness rules, version handling, API contracts, error mapping, or concurrency harnesses. The test suite should cover:

- Basic checkout and order behavior.
- Repeated delivery of one logical completion request.
- Duplicate-order detection at the persistence boundary.
- Replay and loser-reconciliation behavior.
- Version mismatch and stale-write rejection.
- Safe and intentionally unsafe modes.

## Repository Layout

```text
.
├── README.md
├── pyproject.toml
├── uv.lock
├── .env.example
├── demo.py                            # Guided safe demonstration
├── demo_fail.py                       # Intentional unsafe failure demonstration
├── app/
│   ├── main.py                        # FastAPI application factory
│   ├── settings.py                    # SAFETY_MODE and database configuration
│   ├── domain/
│   │   ├── models.py                  # Checkout, order, line item, and version state
│   │   └── errors.py                  # DuplicateOrderError and StateConflictError
│   ├── services/
│   │   └── checkout_service.py        # Completion logic and reconciliation paths
│   ├── infra/
│   │   ├── store.py                   # SQLite schema, unique index, and version checks
│   │   ├── idempotency.py             # Request identity and replay records
│   │   └── lock_manager.py            # Local in-process coordination only
│   ├── routes/
│   │   ├── a2a_rpc.py                 # Agent-to-agent JSON-RPC endpoint
│   │   └── well_known.py              # UCP discovery endpoint
│   └── ucp/                           # UCP constants and schemas
├── experiments/
│   ├── runner.py                      # Concurrency experiment entry point
│   ├── metrics.py                     # Result collection and CSV output
│   └── README.md                      # Experiment instructions
├── scripts/
│   └── generate_plots.py              # Experiment figure generation
├── docs/
│   ├── architecture_system_design.md   # Chapter diagrams and design walkthrough
│   └── plots/                         # Generated figures
└── tests/
```

## Architecture Diagrams and Supporting Documents

`docs/architecture_system_design.md` contains the diagrams used in Chapters 11 through 13:

- Multi-worker deployment topology and the persistence boundary.
- Retry-storm sequence and database uniqueness enforcement.
- Mutation race and version-based stale-write rejection.
- Idempotency fast path and atomic slow path.
- Checkout lifecycle and expected-version checks.
- Loser reconciliation after duplicate-order detection.
- Current limitations in the validation-read and order-insert boundary.

Read the persistence-boundary validation diagram before modifying OCC. The current implementation re-reads the checkout version before order creation, but its validation read and later insert do not yet use one shared transaction. The diagram labels that gap so it remains visible.

## Safety and Operational Limits

- This is a local teaching system. It does not charge cards, contact payment providers, settle money, reserve inventory, or satisfy financial compliance requirements.
- A unique database index prevents duplicate rows at that table boundary. It does not automatically reconcile external side effects that happened before or after the database write.
- The in-memory lock and early replay lookup are optimizations. They are not distributed correctness boundaries.
- The current idempotency record and order record are not shown committing in one shared transaction. A production design must handle in-progress claims, crash recovery, atomic commitment, and reconciliation of external effects.
- The current stale-write check is separated from the later order insert. A stronger OCC implementation performs the version check, order insert, and dependent checkout update in one transaction or guarded write predicate.
- Do not set `SAFETY_MODE=off` outside the isolated failure demonstration.

## Troubleshooting

### A previous run changes the result

Remove the local test database before rerunning a scenario:

```bash
rm -f test.db
```

### The safe demonstration returns an existing order

That can be correct replay behavior. Check the idempotency key, checkout ID, and local database state before treating it as a bug.

### The unsafe demonstration does not produce duplicates

Concurrency failures are scheduling-dependent. Confirm that you are running the intended unsafe mode, using concurrent requests, and not reusing a database that already contains a completed order.

### A multi-worker run behaves differently from the single-process demo

That difference is the lesson. Process-local locks and caches do not cross worker boundaries. Inspect the database constraint and request identity path rather than adding more in-memory locking.

### A stale-write test returns a state conflict

That is the intended guarded outcome. Re-read the checkout state, decide whether the updated version still permits the action, then retry or ask for reconfirmation according to the caller's policy.

## Related Chapters

- Chapter 10 explains why process-local memory cannot provide durable coordination once a service scales across stateless workers.
- Chapter 8 defines the MCP tool boundary, which constrains how a model requests an action but does not provide write safety.
- Chapter 14 adds fail-closed policy gates and human approval holds before sensitive actions execute.
- Chapter 15 monitors optimistic-concurrency conflict rates, validates tool schemas, and turns safety regressions into CI failures.

## License and Errata

See `LICENSE` for licensing terms. Report documentation or code issues through this repository's GitHub issue tracker.
