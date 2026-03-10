# Technical Architecture

The Data Unlock technical architecture is a layered, federated system. It is designed so that each participant operates their own infrastructure while interoperating through shared protocols. There is no single monolithic platform — instead, there are standardised building blocks that can be assembled in different configurations depending on the use case and available technology.

## Architecture Layers

```
┌─────────────────────────────────────────────────────────────────┐
│                           TOOLS                                  │
│        AI Agents, Chatbots, Dashboards, Reports                  │
└─────────────────────────┬───────────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────────┐
│                         SERVICES                                 │
│   APIs & Toolkits for RAG, NLQ, Knowledge Graph, Datasets,      │
│   MCP, Training, Reasoning & Deploying AI Models                 │
└─────────────────────────┬───────────────────────────────────────┘
                          │
┌─────────────┬───────────▼──────────┬────────────────────────────┐
│ DATA SOURCES│   DATA PROCESSING    │    INDEXES & MODELS         │
│             │                      │                             │
│ PDFs, Docs, │ - Ingestion: ETL     │ - Vector DB: embeddings,   │
│ Logs, OLTP, │ - Chunking +         │   metadata                 │
│ OLAP, APIs, │   Embedding          │ - Relational/OLAP: data    │
│ Public      │ - Entity extraction  │   lake/warehouse           │
│ datasets,   │   & linking          │ - Graph DB: Knowledge      │
│ MCP Servers,│ - Metadata & Data    │   Graph                    │
│ Agents      │   Cataloguing        │ - Model Hub: base models,  │
│             │ - Labelling          │   weights                  │
│             │ - Guardrails: PII    │                             │
│             │   redaction, policies│                             │
└─────────────┴──────────────────────┴────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    FOUNDATION LAYER                               │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────┐ ┌────────────┐ │
│  │ Open         │ │ Unified Data │ │ Semantic │ │Observability│ │
│  │ Standards,   │ │ Exchange     │ │Registries│ │Infra-      │ │
│  │ Protocols &  │ │ Rails        │ │& Onto-   │ │structure   │ │
│  │ Specs        │ │              │ │logies    │ │            │ │
│  └──────────────┘ └──────────────┘ └──────────┘ └────────────┘ │
└─────────────────────────────────────────────────────────────────┘

                    ┌──────────────────┐
                    │ Policy &         │
                    │ Contracts        │
                    │ (Governance      │
                    │  Sidecar)        │
                    └──────────────────┘
```

## Layer Descriptions

### Tools Layer
The user-facing interfaces that consume processed data. These include AI agents, chatbots with RAG grounding, analytical dashboards, automated reports, and workflow triggers. Tools are use-case specific and are defined in the [Data Boarding Pass](data-boarding-pass.md).

### Services Layer
Reusable APIs and toolkits that power the Tools layer. This includes RAG pipelines, natural language query engines, knowledge graph query interfaces, dataset access APIs, MCP server endpoints, and model training/inference services. Services are domain-agnostic and shared across use cases.

### Data Layer
The core data infrastructure, split into three components:

**Data Sources** — The raw inputs from DIPs. These may be structured (databases, APIs), semi-structured (CSVs, JSON feeds), or unstructured (PDFs, documents). Data sources also include MCP servers and AI agents from other Data Unlock deployments.

**Data Processing** — The pipeline that transforms raw data into trusted, linked, AI-ready products. This implements the Prepare, Connect, and Enable stages of the value chain. Key operations include ingestion (connectors, ETL), chunking and embedding generation, entity extraction and cross-source linking, metadata cataloguing, labelling (automatic and manual), and guardrails (PII redaction, policy enforcement).

**Indexes & Models** — The persistence and retrieval layer for processed data. Vector databases store embeddings and metadata for similarity search. Relational/OLAP stores (data lake or warehouse) hold cleaned tabular data. Graph databases host the knowledge graph. A model hub stores base models, fine-tuned weights, and model cards.

### Foundation Layer
The cross-cutting infrastructure that every layer depends on:

- **Open Standards, Protocols & Specifications** — The interoperability contracts defined in the [Technical Specifications](../specifications/prepare-layer.md) section.
- **Unified Data Exchange Rails** — The transport layer for moving data between participants (APIs, bulk transfer, streaming).
- **Semantic Registries & Ontologies** — Shared vocabularies, taxonomy servers, and mapping registries that enable semantic alignment.
- **Observability Infrastructure** — Logging, monitoring, audit trails, and compliance dashboards.

### Policy & Contracts (Governance Sidecar)
A parallel governance layer that is not in the data path but governs every interaction. This includes ODRL access policies, consent tokens, data sharing agreements, and audit/compliance services. It operates as a "sidecar" — every data access request is validated against policy before execution.

## Deployment Models

Data Unlock does not prescribe a single deployment topology. Conforming implementations may use:

| Model | Description | Best for |
|---|---|---|
| **Centralised instance** | A single Data Unlocker operates the full stack for all DIPs in a domain. | Small ecosystems, single-agency deployments |
| **Federated mesh** | Each DIP operates their own Prepare layer; a shared Connect/Enable layer federates queries across them. | Multi-agency deployments with strong institutional sovereignty |
| **Hybrid** | Some layers are centralised (e.g., a shared knowledge graph), while others remain federated (e.g., each DIP maintains its own catalogue). | Most real-world deployments |
| **BYOD (Bring Your Own Data)** | The Data Unlocker provides a containerised stack (Docker/Kubernetes) that runs inside the DIP's own cloud. Sensitive data never leaves the firewall. | Deployments with strict data sovereignty requirements |

## Technology Neutrality

The architecture is intentionally technology-neutral. The specifications define **what** must be exposed (e.g., a DCAT-AP compliant catalogue endpoint) but not **how** it must be built internally. A conforming implementation could use:

- PostgreSQL or Snowflake for the relational layer
- Neo4j or Apache Jena for the knowledge graph
- Pinecone, Weaviate, or ChromaDB for the vector store
- Any MCP-compatible server framework for AI agent access
- Google Data Commons, CKAN, or a custom solution for the catalogue

What matters is conformance with the interoperability specifications, not the choice of underlying technology.
