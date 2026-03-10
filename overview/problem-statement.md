# The Problem

## High data availability, low real-world value

Governments worldwide collect enormous volumes of data — census records, trade statistics, agricultural surveys, health indicators, economic metrics, business registrations. Much of this data is technically "open" or at least internally available. Yet the real-world value extracted from it remains remarkably low.

The reason is not a shortage of data. It is a shortage of **data infrastructure** — the pipes, protocols, and trust mechanisms that allow data to flow from where it is collected to where it can generate insight and action.

## Four structural gaps

### 1. Discovery

Data exists, but nobody can find it. Datasets are scattered across departmental websites, internal file servers, PDF reports, and legacy databases. There is no unified catalogue. A researcher looking for district-level agricultural productivity data may not even know which ministry holds it, let alone how to request access.

### 2. Structure

Data lacks context and is not structured for reuse. A CSV exported from one department's system may use column names like `col_a`, `val_1` with no documentation. Date formats vary. Units are unspecified. Without significant manual effort to understand and clean each dataset, it cannot be combined with anything else.

### 3. Silos

Datasets don't link across actors and systems. The Ministry of Commerce tracks exports by HS code. The Ministry of MSME tracks enterprises by Udyam registration number. The Ministry of Statistics tracks indicators by geographic unit. These three datasets contain deeply complementary information, but there is no common identifier, no shared ontology, and no mechanism to join them without a bespoke integration project.

### 4. Machine readiness

Data is human-readable, not AI-ready. A policy document published as a PDF can be read by a human, but an LLM agent cannot reliably extract structured facts from it without first converting it into embeddings, indexing it in a vector store, and mapping its entities to a knowledge graph. Today, every organisation that wants to use AI on government data must build this entire pipeline from scratch.

## The cost of the status quo

These gaps create compounding costs:

- **Duplicated effort**: Every new use case requires a new data pipeline. The same dataset gets cleaned, standardised, and integrated dozens of times by different teams.
- **Delayed decisions**: Policymakers cannot link administrative data to survey data in time to inform live decisions. By the time a cross-ministry dataset is assembled, the policy window has closed.
- **Inaccessible intelligence**: Citizens and businesses cannot "ask the data a question." Access requires deep technical expertise, institutional relationships, and months of data engineering.
- **AI bottleneck**: The promise of AI-driven governance — predictive analytics, conversational access, automated compliance — remains unrealised because the underlying data is not in a form that AI systems can consume.

## What is needed

The solution is not another data portal or analytics dashboard. It is a **shared infrastructure layer** — a set of open specifications and protocols — that makes any participating dataset:

- **Discoverable** — Automatically indexed and searchable across institutions
- **Understandable** — Described by machine-readable schemas with semantic context
- **Linkable** — Connected to related datasets through common identifiers and ontologies
- **Trustworthy** — Certified for quality, governed by explicit access policies, and auditable through provenance credentials
- **AI-ready** — Represented as embeddings, knowledge graph nodes, and vector store entries that AI agents can query directly

This is what Data Unlock provides.
