# Core Building Blocks

> **Prerequisites:** This page assumes familiarity with the [Data Boarding Pass](data-boarding-pass.md) (the deployment manifest for a use case) and the [Data Passport](data-passport.md) (the metadata envelope for a dataset). It also builds on the [Technical Architecture](technical-architecture.md), which defines the federated node structure described below.

Data Unlock's building blocks are organised by **who operates them**. This reflects the federated architecture — there is no single platform that contains all blocks. Each participant type (DIP, Data Unlocker, Nodal Agency) operates its own set of building blocks, and they interoperate through a shared set of protocol blocks.

If you are a DIP, read the DIP blocks — that is what you need to build. If you are a Data Unlocker, read the Data Unlocker blocks. If you are a Nodal Agency, read the Nodal Agency blocks. The protocol blocks are the contracts everyone must follow.

```
┌─────────────────┐   Protocol    ┌──────────────────┐   Protocol    ┌───────────────┐
│  DIP Blocks     │◄─── Blocks ──▶│  Data Unlocker   │◄─── Blocks ──▶│ Nodal Agency  │
│  (per provider) │               │  Blocks          │               │ Blocks        │
│                 │               │  (per integrator)│               │ (one per      │
│                 │               │                  │               │  ecosystem)   │
└─────────────────┘               └──────────────────┘               └───────────────┘
```

---

## 1. DIP Building Blocks

Every Data Information Provider operates these blocks. They run on the DIP's own infrastructure — nothing here requires sharing data with an external system. These blocks make a DIP's data discoverable, trustworthy, and accessible through governed interfaces.

| Block | What it does | Artefact produced |
|---|---|---|
| **Data Catalogue** | Publishes a searchable metadata endpoint describing all available datasets — what exists, what it contains, how fresh it is, how to access it. | DCAT-AP catalogue entries |
| **Schema Registry** | Stores and serves machine-readable structural descriptors for each dataset — field names, types, constraints, relationships. | JSON Schema descriptors, Frictionless Data Package files |
| **Access Policy Engine** | Evaluates every incoming data request against the DIP's defined policies. Determines whether the requester's identity and stated purpose are permitted. Rejects non-compliant requests before any data leaves the DIP. | ODRL policy documents |
| **Provenance & Signing** | Signs each dataset version with a cryptographic hash and issues a verifiable credential attesting to its origin, transformation history, and integrity. | W3C Verifiable Credentials, SHA-256 checksums |
| **Data Quality Service** | Measures and publishes quality metrics (completeness, accuracy, timeliness, consistency) for each dataset. Runs automated checks on each update cycle. | Data Quality Scorecards |
| **Data Passport Publisher** | Assembles and maintains the Data Passport for each data product — combining metadata from the catalogue, schema registry, quality service, policy engine, and provenance signer into a single resolvable document. | Data Passport documents |

### How they fit together inside a DIP

```
                        incoming data request
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Access Policy Engine │──── reject if
                    │ (ODRL evaluation)   │     not permitted
                    └──────────┬──────────┘
                               │ permitted
                               ▼
┌──────────────┐    ┌─────────────────────┐    ┌───────────────────┐
│ Data Quality │    │     Data Store      │    │  Provenance &     │
│ Service      │───▶│   (DIP's own DB,    │───▶│  Signing          │
│              │    │    files, APIs)      │    │  (hash + VC)      │
└──────┬───────┘    └──────────┬──────────┘    └───────┬───────────┘
       │                       │                       │
       ▼                       ▼                       ▼
┌──────────────┐    ┌─────────────────────┐    ┌───────────────────┐
│ Data Passport│◄───│   Schema Registry   │    │  Data Catalogue   │
│ Publisher    │◄───│                     │    │  (DCAT-AP)        │
│              │◄───│                     │    │                   │
└──────────────┘    └─────────────────────┘    └───────────────────┘
```

Every block here runs inside the DIP's own infrastructure. The only external-facing surface is the Catalogue endpoint and the API that responds to data access requests.

---

## 2. Data Unlocker Building Blocks

The Data Unlocker is the integration partner that consumes data from multiple DIPs and transforms it into linked, AI-ready intelligence. These blocks implement the Connect, Enable, and Apply layers of the value chain.

A Data Unlocker pulls data from DIP nodes through their standard APIs (never by direct database access). In BYOD deployments, these blocks run inside the DIP's own cloud — but they are still logically the Data Unlocker's responsibility.

### Connect Blocks

These blocks link data across sources, creating the semantic mesh.

| Block | What it does | Artefact produced |
|---|---|---|
| **Ingestion Pipeline** | Pulls data from multiple DIP endpoints, validates provenance (checks VCs and hashes), and stages it for processing. Handles format normalisation and scheduling. | Ingested, validated data staging area |
| **Entity Resolution Engine** | Matches entities (districts, enterprises, products, people) across datasets from different DIPs using shared identifiers or probabilistic matching. | Entity resolution mapping tables |
| **Semantic Alignment Service** | Maps local field names from each DIP's schema to shared ontologies. Produces JSON-LD context documents that give every field a globally unambiguous meaning. | JSON-LD context documents, vocabulary mappings |
| **Virtual Data Layer** | Routes federated queries across multiple DIP endpoints (or local copies). Decomposes a single query into sub-queries, dispatches them, and assembles the results. | Federated query endpoint (GraphQL or SPARQL) |

### Enable Blocks

These blocks transform linked data into AI-consumable representations.

| Block | What it does | Artefact produced |
|---|---|---|
| **Embedding Pipeline** | Chunks text-heavy data products, generates vector embeddings, and indexes them. Publishes Model Cards documenting which model produced the embeddings. | Embedding indexes, Model Cards |
| **Knowledge Graph** | Represents structured relationships between entities as a traversable graph. Enables questions like "which policies affect HS code 8532 in Karnataka?" | Knowledge Graph (queryable via SPARQL/GraphQL), GraphML exports |
| **Vector Store** | Persists embeddings and supports similarity search. Powers RAG retrieval — given a question, find the most relevant chunks across all ingested data. | Vector search endpoint |
| **Metadata Bridge** | Connects Knowledge Graph entity IDs to Vector Store chunk IDs. The KG knows *what* exists and *how things relate*; the Vector Store knows *what the text says*. The Bridge links them. | KG-to-Vector mapping index |

### Apply Blocks

These blocks deliver intelligence to end users.

| Block | What it does | Artefact produced |
|---|---|---|
| **MCP Server** | Exposes data products as "tools" that AI agents can discover and invoke through the Model Context Protocol. Each tool has a name, description, and parameter schema. | MCP tool definitions, grounded responses with source references |
| **API Gateway** | Provides REST and GraphQL endpoints for programmatic access. Enforces authentication (OAuth 2.0), rate limiting, and policy validation on every request. | OpenAPI specifications |
| **Application Services** | The user-facing interfaces: dashboards for visual analytics, copilots/chatbots for conversational access, AI agents for autonomous multi-step tasks, workflow automation for event-driven actions. | Dashboards, copilot interfaces, agent workflows |
| **Audit Logger** | Records every data access event — who accessed what, for what stated purpose, when, and which policy authorised it. Feeds compliance metrics to the Nodal Agency. | Access logs, compliance reports |

### How they fit together inside a Data Unlocker

```
   DIP A ────┐                                        ┌──── Dashboard
   DIP B ────┤  ┌───────────┐  ┌──────────────┐      ├──── Copilot / Chatbot
   DIP C ────┘  │ Ingestion │  │   Connect    │      ├──── AI Agent
                │ Pipeline  │  │              │      ├──── MCP Server ──── External Agents
                │           │  │ Entity Res.  │      │
     ┌──────────▶ validate  ├─▶│ Semantic     │      │
     │          │ stage     │  │ Alignment    │      │
     │          │           │  │ Virtual DL   │      │
     │          └───────────┘  └──────┬───────┘      │
     │                                │               │
     │                         ┌──────▼───────┐      │
     │                         │   Enable     │      │
     │                         │              │      │
     │                         │ Embedding    │      │
     │                         │ Pipeline     ├──────┘
     │                         │              │   Apply Blocks
     │                         │ Knowledge ◄──┤   (above)
     │                         │ Graph     ├──┤
     │                         │           │  │
     │                         │ Vector  ◄─┤  │
     │                         │ Store   ├─┤  │
     │                         │           │  │
     │                         │ Metadata  │  │
     │                         │ Bridge    │  │
     │                         └──────┬───────┘
     │                                │
     │                         ┌──────▼───────┐
     │                         │ Audit Logger │───▶ Nodal Agency
     │                         └──────────────┘     (compliance metrics)
     │
     │    ┌──────────────┐
     └────│ API Gateway  │◄──── programmatic consumers
          └──────────────┘
```

---

## 3. Nodal Agency Building Blocks

The Nodal Agency is the ecosystem coordinator. It does not sit in the data path — it provides the shared registries, standards, and oversight that enable the federation to function. These blocks are lightweight compared to DIP and Data Unlocker blocks.

| Block | What it does | Artefact produced |
|---|---|---|
| **Boarding Pass Registry** | A searchable index of all active Data Unlock deployments in the ecosystem. Provides visibility into which use cases are live, which DIPs and Data Unlockers are participating, and what interfaces are available. | Registry API, searchable portal |
| **Mapping & Vocabulary Registry** | Stores cross-domain vocabulary mappings (`owl:equivalentClass`, `owl:equivalentProperty`) and canonical reference data (master lists of districts, product codes, industry classifications). Resolves semantic conflicts between DIPs using different ontologies. | Vocabulary mapping files, master data registries |
| **Standards Portal** | Publishes and versions the Data Unlock specifications, JSON Schemas for Data Passport and Data Boarding Pass, and any domain-specific extensions. The single source of truth for what "conformant" means. | Specification documents, JSON Schema files |
| **Compliance Dashboard** | Aggregates compliance metrics from Data Unlocker audit logs (not raw access logs — only summary metrics like access counts, policy violation rates, uptime). Provides oversight without accessing the data itself. | Compliance reports, ecosystem health metrics |
| **Onboarding & Certification** | Manages participant registration, issues DIDs, and verifies conformance through automated checks (e.g., "does this DIP's catalogue endpoint return valid DCAT-AP?"). | Participant registry, conformance certificates |

### What the Nodal Agency does NOT operate

To be explicit: the Nodal Agency does not operate a data catalogue (each DIP has its own), does not store any data products, does not run any knowledge graphs or vector stores, and does not have access to raw audit logs. It is a coordination layer, not a data layer.

---

## 4. Shared Protocol Blocks

These are the interoperability contracts that connect the three participant types. They are not software — they are specifications that every participant must conform to. Think of them as the "rules of the road" for the federation.

| Protocol Block | Between | What it specifies |
|---|---|---|
| **Catalogue Discovery** | Data Unlocker → DIP | How a Data Unlocker discovers what datasets a DIP offers. Standard: DCAT-AP over REST, with support for cross-catalogue harvesting. |
| **Data Access & Transfer** | Data Unlocker → DIP | How data is requested (REST + OAuth 2.0 + JWT with purpose claims), how the DIP validates the request (ODRL evaluation), and how data is transferred (Parquet/CSV + Data Passport reference). |
| **Provenance Verification** | Data Unlocker (internal) | How a Data Unlocker verifies that received data hasn't been tampered with. Standard: validate W3C VC signature + recompute SHA-256 hash. |
| **Federated Query** | Data Unlocker → multiple DIPs | How a single query is decomposed and routed across multiple DIP endpoints. Standard: GraphQL federation or SPARQL SERVICE. |
| **AI Query** | DIU/Agent → Data Unlocker | How an end user or AI agent queries the Data Unlocker's intelligence layer. Standards: MCP (for AI agents), REST/GraphQL (for programmatic access). |
| **Grounded Response** | Data Unlocker → DIU/Agent | How responses include citations tracing every claim back to a specific Data Passport and source dataset. Standard: Citation/Grounding Schema with `source_ref` fields. |
| **Compliance Reporting** | Data Unlocker → Nodal Agency | How aggregated access metrics are reported for oversight. Standard: under development. |
| **Registry Sync** | Nodal Agency → all participants | How vocabulary mappings, standards updates, and boarding pass registrations are distributed. Standard: REST registry API. |

### Why protocols, not a shared bus

In a centralised architecture, you'd have a message bus or API gateway that everything flows through. In Data Unlock's federated architecture, there is no central bus. Each pair of participants communicates directly using the protocol blocks above. This means:

- No single point of failure
- No central chokepoint for data transfer
- No entity that can see all data flows (privacy by design)
- Each participant can scale independently

The trade-off is that each participant must implement the protocol interfaces. The [reference implementations](#what-ships-as-a-specification-vs-software) reduce this burden.

---

## What Ships as a Specification vs. Software?

A key architectural decision is the boundary between **specifications** (open standards everyone conforms to) and **software** (reference implementations published as Digital Public Goods). The federated architecture means this decision is made per-block, not globally.

### DIP Blocks

| Block | Ships as | Rationale |
|---|---|---|
| Data Catalogue | **Both** — spec for the API contract + reference implementation (e.g., CKAN adaptor) | DIPs need a working starting point, but must be free to integrate with existing portals |
| Schema Registry | **Specification** only | JSON Schema is well-understood; no reference implementation needed |
| Access Policy Engine | **Both** — ODRL spec + reference OPA/Rego policy evaluator | Policy evaluation is security-critical; a vetted reference reduces risk |
| Provenance & Signing | **Both** — W3C VC spec + reference signing library | Cryptographic signing is easy to get wrong; a reference library is essential |
| Data Quality Service | **Specification** only (Data Quality Scorecard schema) | Quality measurement is domain-specific; the schema standardises the output format |
| Data Passport Publisher | **Specification** only (Data Passport JSON-LD schema) | The passport is a document format, not a service. Any tool can generate it. |

### Data Unlocker Blocks

| Block | Ships as | Rationale |
|---|---|---|
| Ingestion Pipeline | **Reference implementation** as DPG | Complex ETL with provenance validation; a working codebase accelerates adoption |
| Entity Resolution Engine | **Reference implementation** as DPG | Probabilistic matching + privacy-preserving techniques require careful implementation |
| Semantic Alignment Service | **Both** — JSON-LD spec + reference mapping tool | The spec defines the output; the tool helps non-semantic-web teams produce it |
| Virtual Data Layer | **Reference implementation** as DPG | Federated query routing is architecturally complex |
| Embedding Pipeline | **Reference implementation** as DPG | Chunking strategies, model selection, and indexing are hard to get right from scratch |
| Knowledge Graph | **Reference implementation** as DPG (e.g., Data Commons) | Graph construction and query optimisation benefit enormously from a working starting point |
| Vector Store | **Specification** only (API contract) | Many mature vector DBs exist (Pinecone, Weaviate, Qdrant); no need to build another |
| Metadata Bridge | **Reference implementation** as DPG | Novel component unique to Data Unlock; no existing solution |
| MCP Server | **Reference implementation** as DPG | Enables immediate AI agent connectivity without each implementer building from scratch |
| API Gateway | **Specification** only (OpenAPI contract) | Many mature API gateways exist; the spec ensures interoperability |
| Application Services | **Neither** — use-case specific, not standardised | Dashboards and copilots are built to the Data Boarding Pass; no single reference makes sense |
| Audit Logger | **Both** — log schema spec + reference implementation | The schema ensures interoperability with the Nodal Agency's compliance dashboard |

### Nodal Agency Blocks

| Block | Ships as | Rationale |
|---|---|---|
| Boarding Pass Registry | **Both** — registry API spec + reference implementation | Lightweight service, but needs a standard API for programmatic access |
| Mapping & Vocabulary Registry | **Both** — registry format spec + reference implementation | Needs to be queryable by all participants |
| Standards Portal | **Specification** only | A static website or document repository; no complex software needed |
| Compliance Dashboard | **Reference implementation** as DPG | Aggregation and visualisation of compliance metrics; useful to ship as working software |
| Onboarding & Certification | **Reference implementation** as DPG | Automated conformance checking (e.g., "validate this DCAT-AP endpoint") benefits from shared tooling |

### Protocol Blocks

| Block | Ships as | Rationale |
|---|---|---|
| All protocol blocks | **Specifications** only | These are interoperability contracts — the "rules of the road." They must be open, vendor-neutral, and implementable by anyone. |

### Digital Public Good (DPG) Package

For practical adoption, the reference implementations should be packaged as a **Data Unlock DPG Kit** — a set of containerised, composable services that a DIP or Data Unlocker can deploy and configure. The kit includes:

- **DIP Starter Kit:** Catalogue adaptor, ODRL policy evaluator, VC signing library, Data Passport generator
- **Data Unlocker Kit:** Ingestion pipeline, entity resolution engine, embedding pipeline, knowledge graph (Data Commons), Metadata Bridge, MCP server adaptor
- **Nodal Agency Kit:** Boarding Pass registry, mapping registry, compliance dashboard, conformance checker

Each component in the kit is independently deployable — an implementer can use the full kit or swap individual components with their own implementations, as long as they conform to the protocol specifications.
