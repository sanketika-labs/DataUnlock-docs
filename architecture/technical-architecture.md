# Technical Architecture

> **Prerequisites:** This page assumes familiarity with the [Business Architecture](business-architecture.md) (roles and relationships), the [Data Boarding Pass](data-boarding-pass.md) (deployment manifest), and the [Data Passport](data-passport.md) (dataset metadata envelope).

Data Unlock is a **federated architecture**. There is no central platform that all data flows through. Instead, each participant — every DIP, every Data Unlocker, every DIU — operates its own infrastructure and exposes standardised interfaces. The architecture defines the protocols between participants, not the internals of any single participant.

This is a deliberate design choice. Government data is institutionally owned. A ministry will not (and should not) upload its data to a central system operated by another entity. Data Unlock respects this reality by defining a network of cooperating nodes, not a monolithic platform.

---

## The Federated View

The architecture is best understood as a network of **participant nodes** connected by **shared protocols**. Each node is sovereign — it controls its own data, infrastructure, and access policies. Interoperability comes from conforming to the same protocol layer, not from sharing the same software.

```
                        ┌─────────────────────────┐
                        │     Nodal Agency         │
                        │                          │
                        │  · Boarding Pass Registry │
                        │  · Standards & Specs      │
                        │  · Mapping Registry       │
                        │  · Audit Dashboards       │
                        │                          │
                        │   (coordination only —    │
                        │    no data flows through) │
                        └────────────┬────────────┘
                                     │ governs
                 ┌───────────────────┼───────────────────┐
                 │                   │                    │
    ┌────────────▼──────┐  ┌────────▼─────────┐  ┌──────▼───────────┐
    │     DIP Node A     │  │   DIP Node B      │  │   DIP Node C      │
    │  (e.g., MoSPI)     │  │  (e.g., MoCI)     │  │ (e.g., State Gov) │
    │                    │  │                   │  │                   │
    │ ┌────────────────┐ │  │ ┌───────────────┐ │  │ ┌───────────────┐ │
    │ │ Own Data Store │ │  │ │ Own Data Store│ │  │ │ Own Data Store│ │
    │ └───────┬────────┘ │  │ └──────┬────────┘ │  │ └──────┬────────┘ │
    │         │          │  │        │          │  │        │          │
    │ ┌───────▼────────┐ │  │ ┌──────▼────────┐ │  │ ┌──────▼────────┐ │
    │ │ Prepare Layer  │ │  │ │ Prepare Layer │ │  │ │ Prepare Layer │ │
    │ │ (catalogue,    │ │  │ │               │ │  │ │               │ │
    │ │  schema, policy│ │  │ │               │ │  │ │               │ │
    │ │  provenance)   │ │  │ │               │ │  │ │               │ │
    │ └───────┬────────┘ │  │ └──────┬────────┘ │  │ └──────┬────────┘ │
    │         │ API      │  │        │ API      │  │        │ API      │
    └─────────┼──────────┘  └────────┼──────────┘  └────────┼──────────┘
              │                      │                      │
              │    Shared Protocols: DCAT-AP, JSON Schema,   │
              │    ODRL, W3C VCs, REST/GraphQL               │
              │                      │                      │
    ┌─────────▼──────────────────────▼──────────────────────▼──────────┐
    │                                                                   │
    │                    Data Unlocker Node(s)                          │
    │           (e.g., Google Data Commons, Custom, Thurro)             │
    │                                                                   │
    │   ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
    │   │Connect Layer │  │ Enable Layer │  │   Apply Layer        │  │
    │   │              │  │              │  │                      │  │
    │   │· Entity      │  │· Embeddings  │  │· Dashboards          │  │
    │   │  Resolution  │  │· Vector Store│  │· Copilots / Chatbots │  │
    │   │· Semantic    │  │· Knowledge   │  │· AI Agents           │  │
    │   │  Alignment   │  │  Graph       │  │· Workflow Automation  │  │
    │   │· Virtual     │  │· MCP Server  │  │                      │  │
    │   │  Data Layer  │  │· APIs        │  │                      │  │
    │   └──────────────┘  └──────────────┘  └──────────────────────┘  │
    │                                                                   │
    └──────────────────────────────┬────────────────────────────────────┘
                                   │
                          ┌────────▼────────┐
                          │  DIU / End User  │
                          │  (Policy Maker,  │
                          │   Entrepreneur,  │
                          │   Researcher)    │
                          └─────────────────┘
```

**Key property:** Data does not leave the DIP unless the DIP's access policies allow it. The Data Unlocker either pulls data through governed APIs, or runs inside the DIP's own infrastructure (BYOD model). The Nodal Agency never touches the data — it only coordinates.

---

## What Each Node Operates

### DIP Node

Every Data Information Provider runs its own Prepare layer. This is the minimum participation requirement. A DIP node consists of:

| Component | Responsibility | Runs where |
|---|---|---|
| **Data Store** | The DIP's existing databases, file systems, APIs — unchanged | DIP's own infrastructure |
| **Catalogue Endpoint** | Metadata about available datasets, exposed via a standard API | DIP's own infrastructure |
| **Schema Descriptors** | JSON Schema / Frictionless descriptors for each dataset | Published alongside catalogue |
| **Access Policy Engine** | Evaluates incoming requests against ODRL policies before releasing data | DIP's own infrastructure |
| **Provenance Signer** | Signs each dataset version with a cryptographic credential | DIP's own infrastructure |
| **Data Passport Publisher** | Publishes and maintains Data Passports for each data product | DIP's own infrastructure |

A DIP does not need to install any specific software. It needs to expose standard interfaces. How it does so internally is its own decision.

### Data Unlocker Node

The Data Unlocker is the integration partner that links, enriches, and makes data AI-ready. It operates the Connect, Enable, and (typically) Apply layers. A Data Unlocker node consists of:

| Component | Responsibility |
|---|---|
| **Ingestion Pipeline** | Pulls data from DIP nodes through their APIs, respecting access policies |
| **Entity Resolution Engine** | Matches entities across datasets from different DIPs |
| **Semantic Alignment Service** | Maps local schemas to shared ontologies using JSON-LD contexts |
| **Virtual Data Layer** | Routes federated queries across multiple DIP endpoints |
| **Embedding Pipeline** | Generates vector embeddings from ingested data |
| **Knowledge Graph** | Stores structured relationships between entities across sources |
| **Vector Store** | Stores embeddings for similarity search and RAG |
| **Metadata Bridge** | Connects KG entity IDs to Vector Store chunk IDs |
| **MCP Server** | Exposes data products as tools for AI agent consumption |
| **Application Interfaces** | Dashboards, copilots, chatbots, API endpoints |
| **Audit Logger** | Records all data access events for compliance |

Multiple Data Unlockers can coexist in the same ecosystem. For example, one Data Unlocker might serve MSME export intelligence while another serves policy analytics — both drawing from overlapping DIP nodes.

### Nodal Agency Node

The Nodal Agency is a coordination layer, not a data layer. It never sits in the data path. Its node consists of:

| Component | Responsibility |
|---|---|
| **Boarding Pass Registry** | Searchable index of all active Data Unlock deployments |
| **Mapping Registry** | Cross-domain vocabulary mappings and ontology equivalences |
| **Standards & Specs** | Published specifications that all participants must conform to |
| **Compliance Dashboard** | Aggregated audit data from Data Unlocker nodes (not raw data access logs — only compliance metrics) |
| **Onboarding Service** | Manages participant registration, DID issuance, and conformance verification |

---

## How Nodes Communicate: Protocol Layer

The protocol layer is what makes the federation work. Participants don't share code or infrastructure — they share contracts.

```
┌──────────┐                                          ┌──────────┐
│  DIP A   │◄────── Catalogue Discovery ──────────────│  Data    │
│          │        (DCAT-AP / REST)                   │ Unlocker │
│          │                                          │          │
│          │◄────── Data Access Request ──────────────│          │
│          │        (REST + OAuth 2.0 + purpose claim)│          │
│          │                                          │          │
│          │─────── Policy Validation ────────────────▶│          │
│          │        (ODRL evaluation)                  │          │
│          │                                          │          │
│          │─────── Data Transfer ────────────────────▶│          │
│          │        (Parquet/CSV + Data Passport ref)  │          │
│          │                                          │          │
│          │◄────── Provenance Verification ──────────│          │
│          │        (W3C VC + SHA-256 check)           │          │
└──────────┘                                          └──────────┘

┌──────────┐                                          ┌──────────┐
│  Data    │◄────── AI Query ─────────────────────────│  DIU /   │
│ Unlocker │        (MCP / REST / GraphQL)             │  Agent   │
│          │                                          │          │
│          │─────── Grounded Response ────────────────▶│          │
│          │        (data + source_ref to Passport)    │          │
└──────────┘                                          └──────────┘

┌──────────┐                                          ┌──────────┐
│  Data    │─────── Compliance Metrics ───────────────▶│  Nodal   │
│ Unlocker │        (access counts, policy violations) │  Agency  │
│          │                                          │          │
│          │◄────── Standards / Mappings ──────────────│          │
│          │        (specs, ontology registry)          │          │
└──────────┘                                          └──────────┘
```

### Protocol Summary

| Protocol | Between | Purpose | Standard |
|---|---|---|---|
| **Catalogue Discovery** | Data Unlocker → DIP | Find what datasets exist and evaluate fitness | DCAT-AP over REST |
| **Data Access** | Data Unlocker → DIP | Request data, carrying identity + purpose | REST + OAuth 2.0 + JWT purpose claims |
| **Policy Validation** | DIP (internal) | Evaluate request against ODRL policies | ODRL 2.2 |
| **Data Transfer** | DIP → Data Unlocker | Move data with integrity guarantees | Parquet/CSV + Data Passport sidecar |
| **Provenance Verification** | Data Unlocker (internal) | Verify data hasn't been tampered with | W3C VC + SHA-256 |
| **Federated Query** | Data Unlocker → multiple DIPs | Query across sources in a single request | GraphQL federation / SPARQL SERVICE |
| **AI Query** | DIU/Agent → Data Unlocker | Ask questions, get grounded answers | MCP / REST / GraphQL |
| **Grounded Response** | Data Unlocker → DIU/Agent | Response with citations to source Data Passports | Citation/Grounding Schema |
| **Compliance Reporting** | Data Unlocker → Nodal Agency | Aggregated access metrics for audit | Custom API (not standardised yet) |
| **Standards Distribution** | Nodal Agency → all participants | Publish specs, mappings, registries | Static files / REST registry API |

---

## Deployment Topologies

Because the architecture is federated by design, the same protocol layer supports multiple physical topologies depending on the trust model, data sensitivity, and technical capacity of participants.

### Topology 1: Hub-and-Spoke

One Data Unlocker serves multiple DIPs. The Data Unlocker pulls data from each DIP through their APIs.

```
         DIP A ──────┐
                     │
         DIP B ──────┼──────▶ Data Unlocker ──────▶ DIU
                     │
         DIP C ──────┘
```

**When to use:** A single agency (like MoSPI) sponsors the deployment and a designated Data Unlocker (like Google Data Commons) integrates data from multiple ministries.

**Data sovereignty:** Data leaves the DIP and is processed by the Data Unlocker. Acceptable when DIPs trust the Data Unlocker and policies permit transfer.

### Topology 2: BYOD (Bring Your Own Data)

The Data Unlocker provides a containerised stack (Docker/Kubernetes) that runs *inside* each DIP's infrastructure. The Knowledge Graph and Vector Store sit behind the DIP's firewall. Only query results (not raw data) leave the DIP.

```
    ┌─────────────────────────┐
    │      DIP A's Cloud       │
    │                         │
    │  Data ──▶ [DU Stack]   │──── query results ────▶ DIU
    │          (container)    │
    └─────────────────────────┘

    ┌─────────────────────────┐
    │      DIP B's Cloud       │
    │                         │
    │  Data ──▶ [DU Stack]   │──── query results ────▶ DIU
    │          (container)    │
    └─────────────────────────┘
```

**When to use:** DIPs with strict data sovereignty requirements (e.g., national security data, sensitive personal data). The MoSPI deployment uses this model — the Knowledge Graph sits in MoSPI's own cloud.

**Data sovereignty:** Maximum. Raw data never leaves the DIP's infrastructure.

### Topology 3: Federated Mesh

Multiple Data Unlockers operate independently, each serving a different domain or use case, but they can query each other through the same protocol layer. An MSME Export Unlocker can ask a Trade Statistics Unlocker for tariff data without needing direct DIP access.

```
    DIP A ──▶ Data Unlocker X (MSME) ◄──────────▶ Data Unlocker Y (Trade)  ◄── DIP C
    DIP B ──▶ Data Unlocker X        ◄──────────▶ Data Unlocker Z (Policy) ◄── DIP D
```

**When to use:** Large ecosystems where no single Data Unlocker can (or should) integrate everything. Each domain has its own integration partner, but they federate through shared protocols.

**Data sovereignty:** Each DIP controls which Unlockers it serves. Cross-Unlocker queries use the same access policy validation as DIP-to-Unlocker queries.

### Topology 4: Hybrid

A mix of the above. Some DIPs expose data to a central Unlocker (hub-and-spoke), others run the Unlocker stack internally (BYOD), and some Unlockers federate with each other (mesh). This is the expected topology for most real-world deployments.

---

## What the Architecture Does NOT Centralise

To be explicit about what is federated:

| Component | Centralised? | Where it lives |
|---|---|---|
| Raw data | **No** | Each DIP's own infrastructure |
| Data catalogues | **No** — federated, harvestable | Each DIP publishes its own; Nodal Agency may aggregate an index |
| Access policies | **No** | Each DIP defines and enforces its own |
| Provenance credentials | **No** | Each DIP signs its own |
| Knowledge graph | **Depends on topology** | BYOD: inside DIP. Hub-and-spoke: at Data Unlocker. Mesh: distributed |
| Vector store | **Depends on topology** | Same as above |
| MCP server | **Depends on topology** | Typically at the Data Unlocker, but can run inside the DIP in BYOD |
| Standards & specifications | **Yes** (by design) | Nodal Agency publishes the specs that all participants must conform to |
| Mapping registry | **Yes** (lightweight) | Nodal Agency operates a shared vocabulary mapping service |
| Boarding Pass registry | **Yes** (lightweight) | Nodal Agency maintains the index of active deployments |

The only centralised components are the coordination artefacts (standards, registries, indexes) — never the data or its processing.

---

## Technology Neutrality

The architecture is intentionally technology-neutral. The specifications define **what** each node must expose (e.g., a DCAT-AP catalogue endpoint, an MCP server, a GraphQL query interface) but not **how** it must be built internally. A DIP could be backed by:

- PostgreSQL, Oracle, or flat files for storage
- CKAN, a custom Django app, or a static JSON file for the catalogue
- Any language or framework for the API layer

A Data Unlocker could use:

- Neo4j, Apache Jena, or Google Data Commons for the knowledge graph
- Pinecone, Weaviate, Qdrant, or ChromaDB for the vector store
- Any MCP-compatible server framework for AI agent access
- Any embedding model, provided it publishes a Model Card

What matters is conformance to the protocol layer, not the technology behind it.
