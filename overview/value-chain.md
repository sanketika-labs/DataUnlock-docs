# Value Chain for AI-Ready Data

The Data Unlock value chain defines the four stages that data must pass through to become trusted, linked, machine-readable, and actionable. Each stage addresses a specific class of problem, adds a specific type of readiness, and is backed by concrete technical specifications.

## The Four Stages

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   PREPARE    │───▶│   CONNECT    │───▶│    ENABLE    │───▶│    APPLY     │
│              │    │              │    │              │    │              │
│ Trust the    │    │ Use data     │    │ Make data    │    │ Create       │
│ data         │    │ together     │    │ AI-ready     │    │ outcomes &   │
│              │    │              │    │              │    │ impact       │
├──────────────┤    ├──────────────┤    ├──────────────┤    ├──────────────┤
│ Problem:     │    │ Problem:     │    │ Problem:     │    │ Problem:     │
│ Discovery    │    │ Linkage      │    │ Usability    │    │ Impact       │
├──────────────┤    ├──────────────┤    ├──────────────┤    ├──────────────┤
│ AI Lens:     │    │ AI Lens:     │    │ AI Lens:     │    │ AI Lens:     │
│ Data         │    │ Semantic     │    │ Machine      │    │ Decision     │
│ Intelligence │    │ Intelligence │    │ Readiness    │    │ Intelligence │
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

## Stage 1: Prepare — Trust the Data

**Goal:** Turn fragmented data into standardised, high-quality, governed data products.

Data enters the ecosystem raw — CSVs with ambiguous columns, PDFs without metadata, databases with no documentation. The Prepare stage converts these into trusted data products that any external system can discover, understand, and evaluate for fitness.

| Capability | What it does |
|---|---|
| **Discover** | Creates a centralised catalogue with rich descriptive and structural metadata, data previews, and cross-references. |
| **Understand** | Attaches machine-readable schema definitions (data types, constraints, time-span) so a consuming system can pre-determine fitness for use. |
| **Curate** | Standardises data into open formats (Parquet/Avro) with normalised naming conventions, preventing vendor lock-in. |
| **Authorise** | Encodes access policies as ODRL expressions, enabling purpose-bound access that travels with the data. |
| **Assure** | Signs each dataset version with a cryptographic hash and W3C Verifiable Credential, establishing an immutable provenance trail. |

**Output:** A governed, catalogued, schema-documented data product with verifiable provenance and machine-readable access policies.

[Full technical specification →](../specifications/prepare-layer.md)

---

## Stage 2: Connect — Use Data Together

**Goal:** Resolve entities and align meanings without centralising data.

Individual trusted data products become exponentially more valuable when they can be linked. The Connect stage establishes the semantic bridges that allow a "District" in one dataset to be programmatically linked to the same district in another, without requiring either dataset to change its internal representation.

| Capability | What it does |
|---|---|
| **Entity Resolution** | Matches entities across datasets using Decentralised Identifiers (DIDs) or hashed common keys, without exposing PII. |
| **Semantic Alignment** | Maps local schemas to global ontologies using JSON-LD, ensuring that field names have unambiguous, machine-interpretable meaning. |
| **Federated Query** | Enables cross-source "join" operations through GraphQL or SPARQL endpoints over a Virtual Data Layer. |
| **ML Metadata** | Attaches Croissant metadata descriptors for datasets intended for ML pipeline consumption. |

**Output:** A federated knowledge mesh where datasets from different institutions can be queried and joined as if they were a single source.

[Full technical specification →](../specifications/connect-layer.md)

---

## Stage 3: Enable — Make Data AI-Ready

**Goal:** Represent data in formats optimised for machine reasoning.

Trusted, linked data is still not directly consumable by AI systems. An LLM cannot query a Parquet file. A RAG system cannot ground its responses without embeddings and a vector store. The Enable stage transforms data products into the representations that AI agents, chatbots, and analytics engines actually consume.

| Capability | What it does |
|---|---|
| **Represent** | Generates chunked, embedded representations of data with Model Cards specifying the embedding model, dimensions, and version. |
| **Organise** | Structures data into exportable Knowledge Graphs (GraphML/JSON-LD) and indexes it in vector databases for similarity search. |
| **Access** | Exposes data through governed interfaces: standard APIs, Model Context Protocol (MCP) servers, and Agent-to-Agent (A2A) workflows. |

**Output:** AI-ready data products accessible through MCP servers, knowledge graph endpoints, vector store APIs, and standard REST/GraphQL interfaces.

[Full technical specification →](../specifications/enable-layer.md)

---

## Stage 4: Apply — Create Outcomes and Impact

**Goal:** Convert intelligence into real-world decisions and measurable outcomes.

The final stage is where value is realised. AI-ready data is consumed through interfaces designed for different user personas — policymakers, entrepreneurs, researchers, citizens.

| Capability | What it does |
|---|---|
| **Interact** | Conversational interfaces (copilots, chatbots) with citation/grounding schemas that trace every response back to a specific trusted dataset. |
| **Analyse** | Cross-domain dashboards and statistical analytics powered by OData or similar open analytical APIs. |
| **Decide/Act** | Workflow automation triggered by data events (CloudEvents), enabling proactive alerts, risk models, and recommendation engines. |

**Output:** Dashboards, AI copilots, recommendation engines, and automated workflows — all grounded in trusted, traceable data.

[Full technical specification →](../specifications/apply-layer.md)

---

## Progressive Adoption

A critical property of the value chain is that each stage is independently valuable. An institution does not need to implement all four stages to begin deriving benefits:

| If you implement... | You get... |
|---|---|
| Prepare only | Discoverable, governed, standardised datasets with verifiable provenance |
| Prepare + Connect | Cross-institutional data linking and federated queries |
| Prepare + Connect + Enable | AI-ready data products accessible via MCP, knowledge graphs, and vector stores |
| All four stages | End-to-end intelligence pipeline from raw data to actionable outcomes |
