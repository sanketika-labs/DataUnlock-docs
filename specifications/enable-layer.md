# Enable: Machine Readiness

**Goal:** Transform trusted, linked data into representations optimised for AI consumption — embeddings, knowledge graphs, vector stores, and governed API interfaces.

The Enable layer is where data becomes "AI-ready." After this stage, an LLM agent, a RAG pipeline, or an analytics engine can directly consume the data without requiring bespoke data engineering.

---

## Represent: Embeddings & Chunking

### Specification

Data products that contain text, documents, or semi-structured content **must** be processed into vector embeddings with full provenance metadata.

| Attribute | Requirement |
|---|---|
| **Chunking strategy** | Semantic chunking (paragraph/section-level) preferred over fixed-length. Each chunk must retain a reference to its source document and position. |
| **Embedding model** | Any model, but a **Model Card** must be published for each embedding index |
| **Model Card contents** | Model name, provider, version, dimensions, training data description, known limitations |
| **Versioning** | When the embedding model is updated, a new embedding index must be created. Old indexes must be retained until all consumers have migrated. |
| **Metadata per chunk** | Source document URI, chunk position, Data Passport reference, timestamp |

### Why Model Cards?

Embeddings are not interoperable across different models. A consuming system needs to know which "vector space" produced an embedding to perform meaningful similarity search. Without a Model Card, a consumer cannot determine whether two embedding indexes are comparable.

### Model Card Schema

```json
{
  "@context": "https://dataunlock.org/contexts/model-card/v1",
  "@type": "EmbeddingModelCard",
  "modelName": "text-embedding-3-small",
  "provider": "OpenAI",
  "version": "2024-01",
  "dimensions": 1536,
  "maxTokens": 8191,
  "trainingDataDescription": "Large-scale web corpus (see provider documentation)",
  "knownLimitations": "Performance may degrade on domain-specific government vocabulary",
  "createdAt": "2026-01-15T00:00:00Z",
  "associatedDataProducts": [
    "urn:dataunlock:dp:moci-trade-stats-v2",
    "urn:dataunlock:dp:moci-policy-reg-v1"
  ]
}
```

### Chunk Metadata Schema

```json
{
  "@type": "EmbeddingChunk",
  "chunkId": "urn:dataunlock:chunk:moci-policy-reg-v1:sec3:p2",
  "sourceDocument": "urn:dataunlock:dp:moci-policy-reg-v1",
  "sourceSection": "Section 3: Export Restrictions",
  "sourceParagraph": 2,
  "chunkText": "...",
  "embedding": [0.023, -0.041, ...],
  "modelCardRef": "urn:dataunlock:modelcard:text-embedding-3-small-2024-01",
  "dataPassportRef": "urn:dataunlock:dp:moci-policy-reg-v1",
  "createdAt": "2026-01-15T00:00:00Z"
}
```

### Vector Versioning Policy

When a DIP or Data Unlocker updates the embedding model, all previously generated vectors become obsolete for consumers using the old model. To prevent disruption:

1. A new embedding index is created alongside the old one
2. Both indexes are accessible simultaneously for a migration window (minimum 90 days)
3. The Model Card for the new index is published, and consumers are notified via the catalogue
4. After the migration window, the old index may be deprecated (but should remain accessible in read-only mode)

---

## Organise: Knowledge Graphs

### Specification

Structured and semi-structured data products **should** be represented as knowledge graph nodes and edges, enabling semantic traversal and relationship-based queries.

| Attribute | Requirement |
|---|---|
| **Export format** | [GraphML](http://graphml.graphdrawing.org/) or [JSON-LD](https://www.w3.org/TR/json-ld11/) |
| **Ontology** | Nodes and edges must reference URIs from the semantic context documents (Connect layer) |
| **Exportability** | Knowledge graph "slices" must be exportable so that consuming systems can ingest a subset of the graph for local RAG grounding |
| **Query interface** | SPARQL 1.1 endpoint for graph queries |

### Architecture: Metadata Bridge Pattern

A critical innovation in the Data Unlock architecture is the **Metadata Bridge** — a component that connects the Knowledge Graph to the Vector Store. The Knowledge Graph knows that "Policy #52 exists and applies to HS Code 8532." The Vector Store holds the full text of Policy #52. An AI agent uses the Graph ID to navigate to the correct text in the Vector Store.

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Knowledge Graph  │────▶│ Metadata Bridge   │────▶│  Vector Store   │
│                  │     │                   │     │                 │
│ Entity: Policy52 │     │ Maps KG entity ID │     │ Embedding of    │
│ HS Code: 8532    │     │ to Vector Store   │     │ Policy #52 text │
│ Sector: MSME     │     │ chunk IDs         │     │                 │
└─────────────────┘     └──────────────────┘     └─────────────────┘
```

This pattern enables grounded AI responses — the agent can cite the specific policy document and section that supports its answer, tracing through the Knowledge Graph to the Vector Store to the source document to the Data Passport.

### Knowledge Graph Export Schema

```json
{
  "@context": {
    "schema": "https://schema.org/",
    "du": "https://dataunlock.org/vocab/",
    "lgd": "https://lgdirectory.gov.in/vocab/"
  },
  "@graph": [
    {
      "@id": "du:entity:district:632",
      "@type": "schema:AdministrativeArea",
      "schema:name": "Bangalore Urban",
      "lgd:districtCode": "632",
      "schema:containedInPlace": { "@id": "du:entity:state:29" },
      "du:hasDataProducts": [
        { "@id": "urn:dataunlock:dp:crop-yield-v2" },
        { "@id": "urn:dataunlock:dp:economic-indicators-v3" }
      ]
    },
    {
      "@id": "du:entity:state:29",
      "@type": "schema:State",
      "schema:name": "Karnataka",
      "schema:containedInPlace": { "@id": "du:entity:country:IN" }
    }
  ]
}
```

---

## Access: Governed Interfaces

### Specification

AI-ready data products **must** be accessible through standardised, governed interfaces. Three access methods are specified:

### 1. Standard APIs (REST/GraphQL)

| Attribute | Requirement |
|---|---|
| **API specification** | [OpenAPI 3.1](https://spec.openapis.org/oas/v3.1.0) for REST; GraphQL SDL for graph queries |
| **Authentication** | OAuth 2.0 with JWT bearer tokens |
| **Authorization** | ODRL policy evaluation at the API gateway. Every request must carry purpose and identity claims. |
| **Rate limiting** | Configurable per-consumer rate limits |
| **Versioning** | Semantic versioning in the URL path (e.g., `/api/v2/datasets/...`) |

### 2. Model Context Protocol (MCP)

| Attribute | Requirement |
|---|---|
| **Standard** | [MCP Specification](https://modelcontextprotocol.io/specification) |
| **Purpose** | Expose data products as "tools" that LLM agents can invoke directly |
| **Tool definitions** | Each data product must define MCP tools with clear names, descriptions, and parameter schemas |
| **Grounding** | MCP tool responses must include `source_ref` fields pointing to Data Passport URIs |

MCP provides a universal interface for AI agents to "ask for data," reducing the need for custom API wrappers for every new dataset. An agent connected to a Data Unlock MCP server can discover what data is available, query it, and receive grounded responses — all through a standardised protocol.

#### Example: MCP Tool Definition

```json
{
  "name": "query_trade_statistics",
  "description": "Query India's trade statistics by HS code, country, and time period",
  "parameters": {
    "type": "object",
    "required": ["hs_code"],
    "properties": {
      "hs_code": {
        "type": "string",
        "description": "HS code (2, 4, or 6 digit)"
      },
      "partner_country": {
        "type": "string",
        "description": "ISO 3166-1 alpha-2 country code"
      },
      "year_start": { "type": "integer" },
      "year_end": { "type": "integer" },
      "flow": {
        "type": "string",
        "enum": ["export", "import", "both"]
      }
    }
  },
  "returns": {
    "type": "object",
    "properties": {
      "data": { "type": "array" },
      "source_ref": {
        "type": "string",
        "description": "URI to the Data Passport of the source dataset"
      },
      "freshness": {
        "type": "string",
        "format": "date-time",
        "description": "Timestamp of the most recent data in the response"
      }
    }
  }
}
```

### 3. Agent-to-Agent (A2A) Protocol

| Attribute | Requirement |
|---|---|
| **Standard** | [Google A2A Protocol](https://github.com/google/A2A) (or equivalent) |
| **Purpose** | Enable autonomous agent-to-agent workflows where one Data Unlock deployment's agent delegates tasks to another |
| **Use case** | A trade intelligence agent queries a tariff agent in another jurisdiction, or an analytics agent triggers a data refresh workflow at a DIP |

### 4. Bulk Transfer

| Attribute | Requirement |
|---|---|
| **Format** | Parquet files with Data Passport metadata sidecar |
| **Transport** | SFTP, HTTPS download, or cloud object storage (S3-compatible) |
| **Manifest** | A transfer manifest listing all files, their checksums, and the Data Passport URI |
| **Use case** | Initial data loading, offline analysis, backup |

---

## Open Items

### Vector Drifting

When a DIP updates their embedding model, all previously generated vectors become incompatible with the new model's vector space. The Vector Versioning Policy (above) addresses this operationally, but a formal specification for version negotiation between producer and consumer is still needed.

### API Discovery

There is currently no standardised mechanism for discovering which APIs are available across a Data Unlock ecosystem. A proposed solution is a service registry (similar to a DNS for data APIs) that maps data product URIs to their available access endpoints.

**Proposed approach:** Extend the DCAT-AP catalogue entries with `dcat:endpointURL` and `dcat:endpointDescription` pointing to the OpenAPI/MCP/SPARQL endpoints for each data product.
