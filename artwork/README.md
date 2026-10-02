# 📐 Book Artwork Submission — IEEE-Wiley Production Package

> **Book**: *Building Safe Agentic AI for Enterprise Systems*  
> **Author**: Mohit Aggarwal  
> **Generated**: 2026-09-26 14:23:27  
> **Standards**: [IEEE-Wiley Artwork Guidelines](https://www.wiley.com/publishing/book/steps/prepare-manuscript)

---

## Executive Summary

- **Total Chapters**: 15
- **Total Figures**: 86
- **Submission Files**: 86 TIFF (1200 DPI LZW) + 86 PNG (300 DPI companion) = 172 image files
- **Reference Proofs**: 15 chapter proof PDFs (`c01_proof.pdf` through `c15_proof.pdf`)

---

## Technical Specifications Compliance

| Specification | IEEE-Wiley Requirement | Delivered Package Status |
|---|---|---|
| **Line Art Resolution** | 1200 DPI | ✅ **1200x1200 DPI** across all line art TIFFs |
| **Digital Companion** | 300 DPI | ✅ **300 DPI PNG** companions provided for all figures |
| **Format & Compression** | TIFF with LZW (lossless) | ✅ Tagged Image File Format with LZW compression |
| **Background** | Pure solid white | ✅ Solid white (`#FFFFFF`) with all alpha channels removed |
| **Color Mode** | RGB / Grayscale | ✅ RGB sRGB standard color space |
| **Typography** | Arial / Helvetica 8–12 pt | ✅ Arial / Helvetica sans-serif fonts used throughout |
| **Naming Pattern** | `c{NN}f{NNN}.{ext}` | ✅ Exact pattern: `c01f001.tiff`, `c01f001.png`, etc. |
| **Reference Proofs** | Single PDF per chapter | ✅ 15 per-chapter proof PDFs with cover pages & metadata |

---

## Directory Structure

```
artwork/
├── c01/               # Chapter 1: The Reality of Production Environments (3 figures)
│   ├── c01f001.tiff
│   ├── c01f001.png
│   ├── c01f002.tiff
│   ├── c01f002.png
│   ├── c01f003.tiff
│   ├── c01f003.png
│   └── c01_proof.pdf
├── ...
├── c15/               # Chapter 15: CI/CD for Agentic Systems (15 figures)
│   ├── c15f001.tiff ... c15f015.tiff
│   ├── c15f001.png  ... c15f015.png
│   └── c15_proof.pdf
├── manifest.json      # Machine-readable artwork manifest
└── README.md          # Package overview and index
```

---

## Master Figure Index

| Figure ID | Chapter | Fig # | Source Type | Source Repository | Caption |
|---|---|---|---|---|---|
| `c01f001` | Ch 1 | Fig 1.1 | mermaid | `ch01-production-reality` | The Watchman Trail sequence. The model answers directly from inco... |
| `c01f002` | Ch 1 | Fig 1.2 | mermaid | `ch01-production-reality` | The three possible verdicts. |
| `c01f003` | Ch 1 | Fig 1.3 | mermaid | `ch01-production-reality` | How a question becomes an answer. The AI's opinion is heard but n... |
| `c02f001` | Ch 2 | Fig 2.1 | mermaid | `ch02-agent-monolith-trap` | The monolith write-execute-fix loop assigns parser generation, ex... |
| `c02f002` | Ch 2 | Fig 2.2 | mermaid | `ch02-agent-monolith-trap` | The hand-wired pipeline separates conversion, ledger selection, f... |
| `c02f003` | Ch 2 | Fig 2.3 | mermaid | `ch02-agent-monolith-trap` | The real-statement failure originated in table-structure interpre... |
| `c02f004` | Ch 2 | Fig 2.4 | docx_image | `ch02-agent-monolith-trap` | Sanitized replica of the dense statement layout associated with t... |
| `c02f005` | Ch 2 | Fig 2.5 | mermaid | `ch02-agent-monolith-trap` | The ADK workflow begins after deterministic parsing has returned ... |
| `c02f006` | Ch 2 | Fig 2.6 | docx_image | `ch02-agent-monolith-trap` | Example report generated from categorized transaction data. |
| `c03f001` | Ch 3 | Fig 3.1 | docx_image | `ch03-taming-hallucination-puzzle` | The labeled source document for the chapter’s running query. |
| `c03f002` | Ch 3 | Fig 3.2 | mermaid | `ch03-taming-hallucination-puzzle` | Dense-only retrieval flow. Query and document take the same embed... |
| `c03f003` | Ch 3 | Fig 3.3 | mermaid | `ch03-taming-hallucination-puzzle` | BGE-M3 multi-representation flow. One native model pass produces ... |
| `c03f004` | Ch 3 | Fig 3.4 | mermaid | `ch03-taming-hallucination-puzzle` | Independent hybrid retrieval flow. Dense and lexical retrieval ru... |
| `c03f005` | Ch 3 | Fig 3.5 | mermaid | `ch03-taming-hallucination-puzzle` | Persistent Qdrant retrieval and optional reranking. The first sta... |
| `c04f001` | Ch 4 | Fig 4.1 | mermaid | `ch04-debugging-hallucinations-math` | Root-cause triage for a single observed answer ("Answer states: 9... |
| `c04f002` | Ch 4 | Fig 4.2 | mermaid | `ch04-debugging-hallucinations-math` | The five-layer evaluation stack |
| `c04f003` | Ch 4 | Fig 4.3 | mermaid | `ch04-debugging-hallucinations-math` | The trace-and-score structure for a single evaluation case |
| `c04f004` | Ch 4 | Fig 4.4 | docx_image | `ch04-debugging-hallucinations-math` | A real Opik trace for one evaluation case ("What is the capital e... |
| `c04f005` | Ch 4 | Fig 4.5 | docx_image | `ch04-debugging-hallucinations-math` | The Opik dashboard's case list, filtered to risk:high, showing ei... |
| `c04f006` | Ch 4 | Fig 4.6 | mermaid | `ch04-debugging-hallucinations-math` | The policy gate's six-rule decision cascade |
| `c04f007` | Ch 4 | Fig 4.7 | mermaid | `ch04-debugging-hallucinations-math` | Threshold calibration workflow |
| `c05f001` | Ch 5 | Fig 5.1 | mermaid | `ch05-curing-enterprise-hallucination-crisis` | Adaptive RAG query-routing and self-correction workflow. |
| `c05f002` | Ch 5 | Fig 5.2 | mermaid | `ch05-curing-enterprise-hallucination-crisis` | CRAG decision flow for selecting generation context. |
| `c05f003` | Ch 5 | Fig 5.3 | mermaid | `ch05-curing-enterprise-hallucination-crisis` | SR-RAG critic-and-repair flow for answer quality control. |
| `c05f004` | Ch 5 | Fig 5.4 | mermaid | `ch05-curing-enterprise-hallucination-crisis` | Query Decoupling. |
| `c05f005` | Ch 5 | Fig 5.5 | docx_image | `ch05-curing-enterprise-hallucination-crisis` | Agent trace |
| `c06f001` | Ch 6 | Fig 6.1 | mermaid | `ch06-multimodal-perception` | PerceptionChunk as the grounding contract across all modalities |
| `c06f002` | Ch 6 | Fig 6.2 | mermaid | `ch06-multimodal-perception` | File routing through PerceptionDispatcher |
| `c06f003` | Ch 6 | Fig 6.3 | repo_image | `ch06-multimodal-perception` | Docling layout detection output on a page from patient_care_proto... |
| `c06f004` | Ch 6 | Fig 6.4 | repo_image | `ch06-multimodal-perception` | Sensitivity Scanner redaction running over tabular data content_t... |
| `c06f005` | Ch 6 | Fig 6.5 | repo_image | `ch06-multimodal-perception` | t-SNE projection of CLIP's 512-dimensional dense vectors from the... |
| `c06f006` | Ch 6 | Fig 6.6 | mermaid | `ch06-multimodal-perception` | Cross-Modal Search and Retrieval Workflow. |
| `c07f001` | Ch 7 | Fig 7.1 | mermaid | `ch07-proactive-event-driven-agents` | The extract, transform, load (ETL) ingestion pipeline. |
| `c07f002` | Ch 7 | Fig 7.2 | mermaid | `ch07-proactive-event-driven-agents` | Detailed ETL pipeline showing the geocode cache, incremental hash... |
| `c07f003` | Ch 7 | Fig 7.3 | mermaid | `ch07-proactive-event-driven-agents` | Serial request sequence |
| `c07f004` | Ch 7 | Fig 7.4 | mermaid | `ch07-proactive-event-driven-agents` | Implemented request, state, and telemetry path |
| `c08f001` | Ch 8 | Fig 8.1 | mermaid | `ch08-model-context-protocol` | The problem: no standard boundary. |
| `c08f002` | Ch 8 | Fig 8.2 | mermaid | `ch08-model-context-protocol` | The solution: MCP providing a standard boundary. |
| `c08f003` | Ch 8 | Fig 8.3 | mermaid | `ch08-model-context-protocol` | How FastMCP generates a tool's JSON Schema. |
| `c08f004` | Ch 8 | Fig 8.4 | mermaid | `ch08-model-context-protocol` | Bounded execution in the smart-home server. |
| `c08f005` | Ch 8 | Fig 8.5 | mermaid | `ch08-model-context-protocol` | Safe external API gateway pattern. |
| `c08f006` | Ch 8 | Fig 8.6 | mermaid | `ch08-model-context-protocol` | End-to-end architecture |
| `c08f007` | Ch 8 | Fig 8.7 | mermaid | `ch08-model-context-protocol` | Step-by-step request lifecycle |
| `c09f001` | Ch 9 | Fig 9.1 | docx_image | `ch09-plant-doctor` | The Plant Doctor application. The interface combines visual diagn... |
| `c09f002` | Ch 9 | Fig 9.2 | mermaid | `ch09-plant-doctor` | Enforcing the context boundary. The application halts the workflo... |
| `c09f003` | Ch 9 | Fig 9.3 | user_image | `ch09-plant-doctor` | Plant Doctor enforcing staged inputs. The application splits diag... |
| `c09f004` | Ch 9 | Fig 9.4 | mermaid | `ch09-plant-doctor` | Plant Doctor's dual-interface tool architecture |
| `c09f005` | Ch 9 | Fig 9.5 | mermaid | `ch09-plant-doctor` | Graceful degradation paths. When looking up weather or soil infor... |
| `c09f006` | Ch 9 | Fig 9.6 | mermaid | `ch09-plant-doctor` | Cloud deployment path for Plant Doctor. Containerization builds t... |
| `c10f001` | Ch 10 | Fig 10.1 | mermaid | `ch10-cloud-agent-patterns` | Request 1 arrives at Container A. Container A fills its in-memory... |
| `c10f002` | Ch 10 | Fig 10.2 | mermaid | `ch10-cloud-agent-patterns` | Concurrent Workers Do Not Share Memory. |
| `c10f003` | Ch 10 | Fig 10.3 | mermaid | `ch10-cloud-agent-patterns` | Stateless Agent Delegates Retrieval to Service A. |
| `c10f004` | Ch 10 | Fig 10.4 | mermaid | `ch10-cloud-agent-patterns` | Broken architecture vs. fixed architecture |
| `c10f005` | Ch 10 | Fig 10.5 | mermaid | `ch10-cloud-agent-patterns` | FastAPI Retrieval Service as the Stable Boundary |
| `c10f006` | Ch 10 | Fig 10.6 | mermaid | `ch10-cloud-agent-patterns` | End-to-end request flow on Cloud Run. |
| `c11f001` | Ch 11 | Fig 11.1 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Deployment Topology (Multi-Worker). Illustrates why in-memory loc... |
| `c11f002` | Ch 11 | Fig 11.2 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Retry Storm Sequence (Payment Integrity). Shows how repeated deli... |
| `c11f003` | Ch 11 | Fig 11.3 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Mutation Race Sequence (Optimistic Concurrency) shows how a payme... |
| `c12f001` | Ch 12 | Fig 12.1 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Retry-safe checkout completion across concurrent workers. Stable ... |
| `c12f002` | Ch 12 | Fig 12.2 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Fast path versus atomic slow path |
| `c12f003` | Ch 12 | Fig 12.3 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | The schema settles the race |
| `c12f004` | Ch 12 | Fig 12.4 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Commit authority stays below the model boundary. |
| `c13f001` | Ch 13 | Fig 13.1 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Checkout version lifecycle. |
| `c13f002` | Ch 13 | Fig 13.2 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | OCC validation gate at the persistence boundary. |
| `c13f003` | Ch 13 | Fig 13.3 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Persistence-boundary validation in the repository. |
| `c13f004` | Ch 13 | Fig 13.4 | mermaid | `ch11-13-ucp-a2a-payment-integrity` | Conflict recovery in complete_checkout() |
| `c14f001` | Ch 14 | Fig 14.1 | mermaid | `ch14-enterprise-ai-assistant` | SentinelAI&apos;s SecurityPipeline runs pre- and post-large langu... |
| `c14f002` | Ch 14 | Fig 14.2 | mermaid | `ch14-enterprise-ai-assistant` | Pre-inference scope enforcement in pipeline.py. |
| `c14f003` | Ch 14 | Fig 14.3 | mermaid | `ch14-enterprise-ai-assistant` | Post-inference output pipeline in pipeline.py |
| `c14f004` | Ch 14 | Fig 14.4 | mermaid | `ch14-homecare-visit-triage-agent` | The Homecare agent’s LangGraph state machine |
| `c14f005` | Ch 14 | Fig 14.5 | mermaid | `ch14-homecare-visit-triage-agent` | Real patient filename flows through the Homecare agent. |
| `c15f001` | Ch 15 | Fig 15.1 | mermaid | `ch15-agentic-cicd` | The eval-gate merge gate. A pull request triggers the offline eva... |
| `c15f002` | Ch 15 | Fig 15.2 | docx_image | `ch15-agentic-cicd` | Eval-gate feedback in a pull request. The workflow posts the eval... |
| `c15f003` | Ch 15 | Fig 15.3 | docx_image | `ch15-agentic-cicd` | Prompt regression caught before merge. |
| `c15f004` | Ch 15 | Fig 15.4 | mermaid | `ch15-agentic-cicd` | The model migration gate. Cached candidate traces are replayed th... |
| `c15f005` | Ch 15 | Fig 15.5 | docx_image | `ch15-agentic-cicd` | A candidate blocked despite acceptable aggregate deltas. |
| `c15f006` | Ch 15 | Fig 15.6 | docx_image | `ch15-agentic-cicd` | Optimistic-concurrency retry drift. |
| `c15f007` | Ch 15 | Fig 15.7 | docx_image | `ch15-agentic-cicd` | Retrieval-similarity drift |
| `c15f008` | Ch 15 | Fig 15.8 | mermaid | `ch15-agentic-cicd` | Live telemetry and drift detection. Evaluation events flow to the... |
| `c15f009` | Ch 15 | Fig 15.9 | docx_image | `ch15-agentic-cicd` | Standard red-team suite output |
| `c15f010` | Ch 15 | Fig 15.10 | docx_image | `ch15-agentic-cicd` | Terminal output, Layer 7 fault-injection run. |
| `c15f011` | Ch 15 | Fig 15.11 | mermaid | `ch15-agentic-cicd` | Automated red-teaming in the CI pipeline. The suite sends malicio... |
| `c15f012` | Ch 15 | Fig 15.12 | docx_image | `ch15-agentic-cicd` | Terminal output, HITL override capture. |
| `c15f013` | Ch 15 | Fig 15.13 | docx_image | `ch15-agentic-cicd` | Terminal output, threshold calibration. |
| `c15f014` | Ch 15 | Fig 15.14 | mermaid | `ch15-agentic-cicd` | The HITL feedback loop. Reviewer assertions become provisional go... |
| `c15f015` | Ch 15 | Fig 15.15 | mermaid | `ch15-agentic-cicd` | Day 2 merge gate overview. Code, prompt, and model changes enter ... |

---

## Submission Checklist for Production Editor

- [x] All 86 figures exported as standalone TIFF files at 1200 DPI
- [x] Lossless LZW compression applied to eliminate compression artifacts
- [x] No alpha channels or transparency (solid white `#FFFFFF` throughout)
- [x] All 86 companion PNG files generated at 300 DPI for digital inspection
- [x] 15 per-chapter consolidated proof PDFs (`c01_proof.pdf` .. `c15_proof.pdf`)
- [x] Manifest file `manifest.json` indexing all technical parameters and hashes
- [x] Strict figure numbering `c{NN}f{NNN}` matching Wiley manuscript conventions