# Interoperability Standards

This page provides a consolidated reference of all standards, protocols, and specifications referenced across the Data Unlock technical architecture. Implementers can use this as a checklist to ensure their deployment conforms to the required interoperability contracts.

## Standards by Category

### Metadata & Semantics

| Standard | Version | Purpose in Data Unlock | Layer | Status |
|---|---|---|---|---|
| [DCAT-AP](https://semiceu.github.io/DCAT-AP/releases/3.0.0/) | 3.0 | Metadata catalogue format for data product discovery | Prepare | **Required** |
| [JSON Schema](https://json-schema.org/specification) | 2020-12 | Machine-readable structural descriptors for data products | Prepare | **Required** |
| [Frictionless Data Package](https://specs.frictionlessdata.io/data-package/) | 1.0 | Contextual data packaging (licensing, sources, schema) | Prepare | **Required** |
| [JSON-LD](https://www.w3.org/TR/json-ld11/) | 1.1 | Semantic annotation and context mapping | Connect | **Required** |
| [Schema.org](https://schema.org/) | latest | General-purpose vocabulary for semantic alignment | Connect | **Required** |
| [Croissant](https://mlcommons.org/croissant/) | 1.0 | ML-ready dataset metadata | Connect | Recommended |
| [SKOS](https://www.w3.org/TR/skos-reference/) | 1.0 | Vocabulary and taxonomy management | Connect | Recommended |
| [OWL 2](https://www.w3.org/TR/owl2-overview/) | 2.0 | Ontology mapping and equivalence declarations | Connect | Recommended |

### Data Formats

| Standard | Purpose in Data Unlock | Layer | Status |
|---|---|---|---|
| [Apache Parquet](https://parquet.apache.org/) | Primary open format for tabular data products | Prepare | **Required** |
| [Apache Avro](https://avro.apache.org/) | Streaming/event data format | Prepare | Recommended |
| [GraphML](http://graphml.graphdrawing.org/) | Knowledge graph export format | Enable | **Required** (for KG) |
| [CSV (RFC 4180)](https://tools.ietf.org/html/rfc4180) | Secondary distribution format | Prepare | Recommended |

### Access & APIs

| Standard | Version | Purpose in Data Unlock | Layer | Status |
|---|---|---|---|---|
| [OpenAPI](https://spec.openapis.org/oas/v3.1.0) | 3.1 | REST API specification | Enable | **Required** |
| [GraphQL](https://graphql.org/) | latest | Graph query interface, federated queries | Connect, Enable | **Required** |
| [SPARQL](https://www.w3.org/TR/sparql11-query/) | 1.1 | RDF/knowledge graph queries | Connect, Enable | Recommended |
| [OData](https://www.odata.org/documentation/) | 4.01 | Analytical API queries | Apply | Recommended |
| [MCP](https://modelcontextprotocol.io/specification) | latest | AI agent data access interface | Enable | **Required** |
| [A2A](https://github.com/google/A2A) | latest | Agent-to-agent workflows | Enable | Recommended |
| [OAuth 2.0](https://oauth.net/2/) | 2.0 | API authentication | Enable | **Required** |

### Trust & Compliance

| Standard | Version | Purpose in Data Unlock | Layer | Status |
|---|---|---|---|---|
| [ODRL](https://www.w3.org/TR/odrl-model/) | 2.2 | Access policy encoding | Prepare | **Required** |
| [W3C Verifiable Credentials](https://www.w3.org/TR/vc-data-model-2.0/) | 2.0 | Provenance and integrity attestation | Prepare | **Required** |
| [W3C DIDs](https://www.w3.org/TR/did-core/) | 1.0 | Decentralised institutional identifiers | Connect | **Required** |
| [CloudEvents](https://cloudevents.io/) | 1.0 | Event-driven workflow triggers | Apply | Recommended |
| [OPA/Rego](https://www.openpolicyagent.org/) | latest | Runtime policy evaluation | Enable | Recommended |

### Encoding & Identifiers

| Standard | Purpose in Data Unlock | Status |
|---|---|---|
| UTF-8 | Text encoding for all data products | **Required** |
| ISO 8601 | Date and time formatting | **Required** |
| ISO 3166 | Country and subdivision codes | **Required** |
| ISO 4217 | Currency codes | **Required** |
| HS Nomenclature (WCO) | Product classification for trade data | Domain-specific |
| SDMX | Statistical data exchange | Domain-specific |
| ISIC Rev.4 | Industry classification | Domain-specific |

---

## Conformance Levels

Data Unlock defines three conformance levels, allowing progressive adoption:

### Level 1: Discoverable (Prepare layer)

The minimum viable conformance. A DIP at Level 1 has:

- A DCAT-AP compliant metadata catalogue endpoint
- JSON Schema descriptors for each data product
- Data in Parquet or CSV format with standardised naming
- ODRL access policies
- W3C Verifiable Credentials for provenance

**Value delivered:** Datasets are discoverable, understandable, and governed. Other institutions can find and evaluate them.

### Level 2: Linked (Prepare + Connect layers)

Building on Level 1, a DIP at Level 2 additionally has:

- JSON-LD semantic context documents for each data product
- Entity resolution using DIDs or hashed common keys
- GraphQL or SPARQL query endpoint for federated queries
- Participation in the Cross-Domain Mapping Registry

**Value delivered:** Datasets can be linked and queried across institutions without bespoke integration.

### Level 3: AI-Ready (Prepare + Connect + Enable + Apply layers)

The full conformance level. A deployment at Level 3 additionally has:

- Embeddings with Model Cards and vector store endpoints
- Knowledge graph representation with SPARQL or GraphQL access
- MCP server exposing data products as agent-accessible tools
- Citation/grounding schemas in all conversational and analytical interfaces
- Audit logging for all data access events

**Value delivered:** Full AI-ready data infrastructure with conversational access, automated workflows, and complete traceability.

---

## Compatibility with Existing Standards Ecosystems

Data Unlock does not invent new standards where existing ones suffice. The table below maps Data Unlock requirements to established standards ecosystems:

| Data Unlock requirement | Existing ecosystem | Relationship |
|---|---|---|
| Data cataloguing | European Open Data / data.gov portals | Data Unlock adopts DCAT-AP, the same standard used by these portals |
| Semantic interoperability | Linked Open Data / Semantic Web | Data Unlock uses JSON-LD and Schema.org from this ecosystem |
| Access policies | Rights Expression Languages | Data Unlock uses ODRL from the W3C |
| Provenance | Self-Sovereign Identity / VCs | Data Unlock uses W3C VCs and DIDs |
| AI agent access | Anthropic MCP / Google A2A | Data Unlock adopts these emerging standards directly |
| ML readiness | MLCommons | Data Unlock uses Croissant from MLCommons |
| Statistical data | SDMX (UN, OECD, Eurostat) | Data Unlock is compatible with SDMX and can bridge legacy SDMX systems to modern AI-ready interfaces |
