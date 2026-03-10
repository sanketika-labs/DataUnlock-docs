# Compliance Checklist

Use this checklist to verify that your Data Unlock deployment conforms to the specification. Items are organised by conformance level.

---

## Level 1: Discoverable (Prepare Layer)

### Catalogue & Discovery

- [ ] DCAT-AP 3.0 compliant catalogue endpoint is deployed and accessible via HTTPS
- [ ] Catalogue supports OAI-PMH or CSW harvesting for automated cross-institution indexing
- [ ] Each data product has a catalogue entry with: title, description, publisher, temporal coverage, spatial coverage, keywords, theme, and at least one distribution
- [ ] Data previews (first 100 rows in CSV) are available for each tabular dataset

### Schema & Structure

- [ ] Every data product has a JSON Schema (2020-12) descriptor
- [ ] Every data product has a Frictionless Data Package descriptor (`datapackage.json`)
- [ ] Schema includes: data types, constraints (min/max, required, enum), and descriptions for every field
- [ ] All field names follow `snake_case` convention
- [ ] All dates use ISO 8601 format
- [ ] All text is encoded in UTF-8
- [ ] Geographic identifiers use ISO 3166 or equivalent national registry
- [ ] Currency values use ISO 4217 codes

### Data Formats

- [ ] Data is available in Apache Parquet (primary format)
- [ ] CSV is available as a secondary distribution format
- [ ] Null values are represented as explicit nulls (not empty strings, "N/A", or "-")

### Access Policies

- [ ] Every data product has an ODRL 2.2 policy document
- [ ] Every permission includes a `purpose` constraint
- [ ] Policies are encoded in JSON-LD format
- [ ] Policy documents are referenced from the Data Passport

### Provenance & Trust

- [ ] The publishing institution has a W3C Decentralised Identifier (DID)
- [ ] Every dataset version is signed with a SHA-256 integrity hash
- [ ] A W3C Verifiable Credential attests to the provenance of each dataset version
- [ ] The VC includes lineage information (sources, transformations, agents)
- [ ] The proof uses Ed25519Signature2020 or JsonWebSignature2020

### Data Passport

- [ ] Every data product has a Data Passport document
- [ ] Data Passport includes: Identity, Discovery, Structure, Quality, Provenance, and Policy sections
- [ ] Data Passport is resolvable via its URI

---

## Level 2: Linked (Prepare + Connect Layers)

*All Level 1 requirements, plus:*

### Entity Resolution

- [ ] Common entities across datasets are identified and mapped to canonical identifiers
- [ ] Identifiers use W3C DIDs (for institutions) or hashed common keys (for sensitive entities)
- [ ] An entity resolution mapping document exists for each cross-dataset link
- [ ] PII is never transmitted in plaintext for matching purposes

### Semantic Alignment

- [ ] Every data product has a JSON-LD 1.1 context document
- [ ] Every field in every dataset maps to a URI from a recognised vocabulary
- [ ] Schema.org is used as the primary vocabulary, supplemented by domain-specific ontologies
- [ ] Fields that do not match existing vocabulary terms are registered in a documented extension vocabulary

### Federated Query

- [ ] A GraphQL or SPARQL endpoint is deployed for federated queries
- [ ] The endpoint supports queries spanning multiple data products
- [ ] Authentication uses OAuth 2.0 with JWT bearer tokens
- [ ] Each DIP validates access policies independently for federated query sub-requests

### Cross-Domain Mapping

- [ ] Vocabulary mappings are registered in the Cross-Domain Mapping Registry (if applicable)
- [ ] `owl:equivalentClass` and `owl:equivalentProperty` assertions are documented

### Data Passport Updates

- [ ] Data Passports include the Semantics section
- [ ] Semantic context URIs are resolvable

---

## Level 3: AI-Ready (Full Stack)

*All Level 1 and Level 2 requirements, plus:*

### Embeddings & Vector Store

- [ ] Text-heavy data products are chunked and embedded
- [ ] Each embedding index has a published Model Card (model name, provider, version, dimensions)
- [ ] Each chunk retains a reference to its source document, position, and Data Passport
- [ ] A vector store endpoint is deployed and accessible via API
- [ ] Vector Versioning Policy is documented and operational

### Knowledge Graph

- [ ] Structured data is represented as knowledge graph nodes and edges
- [ ] KG is exportable in GraphML or JSON-LD format
- [ ] A SPARQL or GraphQL endpoint exposes the knowledge graph
- [ ] The Metadata Bridge connects KG entities to Vector Store chunks

### MCP Server

- [ ] An MCP server is deployed exposing data products as tools
- [ ] Each MCP tool has a clear name, description, and parameter schema
- [ ] MCP tool responses include `source_ref` fields pointing to Data Passport URIs

### API Access

- [ ] REST APIs are documented with OpenAPI 3.1 specifications
- [ ] APIs enforce ODRL policies at the gateway (purpose + identity validation)
- [ ] APIs support semantic versioning in the URL path

### Interfaces & Grounding

- [ ] All conversational interfaces implement the Citation/Grounding Schema
- [ ] Every factual claim includes a `source_ref` to a Data Passport URI
- [ ] Users can "drill down" from a citation to the source data product
- [ ] Analytical dashboards expose data via OData or GraphQL analytical APIs

### Audit & Compliance

- [ ] All data access events are logged (timestamp, requester, purpose, data products accessed)
- [ ] Consent/policy validation is recorded in the audit log for every access
- [ ] Audit logs are retained for a minimum of 1 year (or as specified by the Nodal Agency)
- [ ] The Nodal Agency has read access to aggregated audit dashboards

### Data Boarding Pass

- [ ] A complete Data Boarding Pass exists for the deployment
- [ ] The Boarding Pass is registered with the Nodal Agency
- [ ] All referenced Data Passports are resolvable

### Workflow Automation (if applicable)

- [ ] Events use CloudEvents v1.0 format
- [ ] Standard event types are used (`dataunlock.data.published`, `dataunlock.alert.threshold`, etc.)
- [ ] Event consumers validate data access policies before processing

---

## Self-Assessment Scoring

For each checklist item, score your deployment:

| Score | Meaning |
|---|---|
| **2** | Fully implemented and operational |
| **1** | Partially implemented or planned |
| **0** | Not implemented |

**Level 1 target:** All Level 1 items score 2.
**Level 2 target:** All Level 1 + Level 2 items score 2.
**Level 3 target:** All items score 2.

Report your self-assessment to the Nodal Agency as part of onboarding and annual compliance reviews.
