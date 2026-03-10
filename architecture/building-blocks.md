# Core Building Blocks

Data Unlock defines four categories of building blocks. Each building block is a distinct capability that can be implemented independently, but which integrates with others through the standardised interfaces defined in the [Technical Specifications](../specifications/prepare-layer.md).

Implementers should think of these as the "services" that a Data Unlock deployment must provide. The specifications section defines the protocols; this page defines the **capabilities**.

## 1. Trust & Governance Services

These services establish the foundation of trust that every other layer depends on.

| Service | Responsibility | Key Standards |
|---|---|---|
| **Data Catalogue & Inventory** | Maintains a searchable registry of all data products in the ecosystem, with rich descriptive and structural metadata. | DCAT-AP, JSON Schema, Frictionless Data Packages |
| **Consent, Licensing & Audit** | Manages purpose-bound access policies, tracks consent tokens, and maintains an immutable audit log of all data access events. | ODRL 2.2, W3C Verifiable Credentials |
| **Metadata, Quality & Sensitivity** | Stores and serves metadata about data quality scores, sensitivity classifications (PII, confidential, public), and freshness indicators. | ISO 8000, custom Data Quality Scorecard schema |
| **Privacy-Preserving Models** | Provides capabilities for accessing data insights without exposing raw data — differential privacy, synthetic data generation, secure multi-party computation. | OpenDP, PySyft (reference implementations) |

### Artefacts produced
- Data Quality Scorecard (per dataset)
- ODRL policy documents (per dataset)
- W3C Verifiable Credentials (per dataset version)
- Catalogue entries (DCAT-AP records)

---

## 2. Semantic & Linking Services

These services create the "mesh" that connects individual data products into a coherent, queryable knowledge layer.

| Service | Responsibility | Key Standards |
|---|---|---|
| **Entity & Identifier Resolution** | Matches entities (districts, enterprises, products, people) across datasets using shared identifiers or probabilistic matching. | W3C DIDs, SHA-256 hashed keys |
| **Schema & Definition Alignment** | Maps local field names and data types to shared global ontologies, resolving vocabulary differences. | JSON-LD, Schema.org, domain-specific ontologies |
| **Reference & Master Data** | Maintains canonical lists (districts, HS codes, industry classifications) that serve as the "join keys" across datasets. | ISO 3166, UN/CEFACT, domain registries |
| **Semantic Metadata** | Stores and serves the JSON-LD context documents, ontology mappings, and vocabulary registries that enable semantic interoperability. | JSON-LD 1.1, OWL 2, SKOS |

### Artefacts produced
- JSON-LD context documents (per data product)
- Entity resolution mapping tables
- Cross-domain vocabulary mapping registry
- Master data registries

---

## 3. AI-Readiness Services

These services transform trusted, linked data into the representations that AI systems consume.

| Service | Responsibility | Key Standards |
|---|---|---|
| **Chunking, Embeddings & Features** | Splits documents and datasets into semantic chunks, generates vector embeddings, and extracts features for ML pipelines. | Model Cards (for embedding provenance), Croissant (for ML metadata) |
| **Indexing & Similarity Search** | Maintains vector databases that enable similarity-based retrieval for RAG systems and semantic search. | No single standard; interop via API contracts |
| **Vector Databases & Knowledge Graphs** | Operates the persistence layer for embeddings (vector DB) and structured relationships (knowledge graph). | GraphML, JSON-LD (for KG export), OpenAPI (for vector DB APIs) |
| **APIs, MCP, NLQ, RAG & Models** | Exposes AI-ready data through standard interfaces: REST APIs, MCP servers for AI agent consumption, natural language query endpoints, and RAG pipeline APIs. | MCP specification, OpenAPI 3.1, A2A protocol |

### Artefacts produced
- Embedding indexes with Model Cards
- Knowledge graph exports (GraphML/JSON-LD)
- MCP server tool definitions
- Croissant metadata descriptors (for ML datasets)

---

## 4. Access & Interface Services

These services deliver the "last mile" — converting AI-ready data into interfaces that humans and machines consume.

| Service | Responsibility | Key Standards |
|---|---|---|
| **Dashboards & Reports** | Visual analytics and statistical reporting interfaces for monitoring, trend detection, and policy oversight. | OData (for analytical APIs), CloudEvents (for real-time updates) |
| **Chatbots, RAG Assistants & NLQ** | Conversational AI interfaces grounded in the knowledge graph and vector stores, with citation schemas tracing responses to source data. | Citation/Grounding Schemas, MCP |
| **AI Agents** | Autonomous or semi-autonomous agents that perform multi-step tasks (research, analysis, compliance checking) using the Data Unlock infrastructure. | A2A protocol, MCP, Tool/Function calling |

### Artefacts produced
- Dashboard configurations
- Agent workflow definitions
- Citation/grounding schema documents

---

## What ships as a Specification vs. Software?

A key architectural decision in Data Unlock is the boundary between **specifications** (open standards that anyone must conform to) and **software** (reference implementations published as Digital Public Goods).

| Component | Manifest as | Rationale |
|---|---|---|
| **Data Boarding Pass** | **Specification** (JSON Schema) | It is a declarative manifest — a "deployment descriptor." Multiple tools can generate and consume it. |
| **Data Passport** | **Specification** (JSON-LD envelope schema) | It is a metadata standard. Any system that handles data must be able to read and write it. |
| **Interoperability protocols** (DCAT-AP catalogues, ODRL policies, MCP interfaces, etc.) | **Specifications** | These are the "rules of the road" that enable interoperability. They must be open and vendor-neutral. |
| **Data Catalogue service** | **Both** — specification for the API contract + reference implementation as a DPG | Implementers need a working starting point, but must be free to swap in alternatives. |
| **Knowledge Graph engine** | **Reference implementation** as a DPG | Complex enough that a working codebase dramatically accelerates adoption. |
| **Embedding pipeline** | **Reference implementation** as a DPG | Same rationale — the specification alone is not sufficient for practical adoption. |
| **MCP server adaptor** | **Reference implementation** as a DPG | Enables immediate AI agent connectivity without each implementer building from scratch. |
| **Consent & audit service** | **Both** — specification + reference implementation | The specification ensures interoperability; the reference implementation ensures security best practices. |
