# Getting Started

This guide walks a government agency or IT vendor through the process of planning and deploying a Data Unlock implementation — from initial assessment to a functioning deployment.

## Prerequisites

Before beginning, ensure you have:

1. **A defined use case.** Data Unlock is purpose-driven. You need a specific problem statement (e.g., "Enable policymakers to monitor development indicators across ministries in real-time").
2. **Identified data sources.** You need at least two data sources that would benefit from being linked and made AI-ready.
3. **Institutional support.** A Sponsor and a Nodal Agency (or equivalent authority) willing to coordinate the effort.
4. **Technical capacity.** A team capable of implementing APIs, data pipelines, and (for Level 3) knowledge graphs and embedding infrastructure.

## Implementation Roadmap

### Phase 1: Assessment & Planning

**Objective:** Understand the current state of your data and define what success looks like.

**Step 1: Data Inventory**

Catalogue all datasets relevant to your use case. For each dataset, document:

- Current format and storage location
- Approximate size and update frequency
- Data owner and governance status
- Known quality issues
- Current access mechanisms (API, file download, database, etc.)

**Step 2: Gap Analysis**

For each dataset, assess readiness against the four Data Unlock stages:

| Question | Stage | If "No" → |
|---|---|---|
| Is the dataset described in a machine-readable catalogue? | Prepare | Need DCAT-AP catalogue setup |
| Does it have a JSON Schema descriptor? | Prepare | Need schema documentation |
| Is it available in Parquet/Avro/CSV? | Prepare | Need format conversion |
| Are access policies documented and machine-readable? | Prepare | Need ODRL policy encoding |
| Is provenance verifiable with cryptographic signatures? | Prepare | Need W3C VC implementation |
| Are entities resolvable across datasets? | Connect | Need entity resolution |
| Are field meanings mapped to shared ontologies? | Connect | Need JSON-LD context creation |
| Are embeddings generated with Model Cards? | Enable | Need embedding pipeline |
| Is there a knowledge graph representation? | Enable | Need KG construction |
| Is data accessible via MCP server? | Enable | Need MCP implementation |

**Step 3: Draft the Data Boarding Pass**

Using the [Data Boarding Pass schema](../architecture/data-boarding-pass.md), create an initial deployment manifest that defines your sector, use case, data sources, beneficiaries, and planned interfaces. This serves as the project charter.

**Step 4: Define roles**

Map your organisations to the [business architecture roles](../architecture/business-architecture.md): Sponsor, Problem Curator, DIPs, Data Unlocker, DIUs, Nodal Agency.

### Phase 2: Prepare Layer Implementation

**Objective:** Make all participating datasets discoverable, governed, and trustworthy.

For each DIP:

1. **Standardise formats.** Convert datasets to Parquet (primary) with CSV as secondary. Enforce snake_case naming, UTF-8 encoding, ISO 8601 dates.

2. **Create schema descriptors.** Write JSON Schema and Frictionless Data Package descriptors for each dataset. This is often the most time-consuming step — it requires understanding every field in the dataset.

3. **Deploy catalogue endpoint.** Set up a DCAT-AP compliant catalogue. Options include CKAN (open-source), custom implementation, or a managed service. Ensure the catalogue supports OAI-PMH or CSW harvesting.

4. **Encode access policies.** Convert existing access rules into ODRL expressions. Start simple — even a basic "permitted for research purposes" policy is better than nothing.

5. **Implement provenance signing.** Generate a DID for each DIP. Sign each dataset version with a W3C Verifiable Credential containing the SHA-256 hash and lineage description.

6. **Create Data Passports.** For each dataset, create an initial Data Passport (Identity, Discovery, Structure, Quality, Provenance, Policy sections).

**Deliverable:** Level 1 conformance. Datasets are discoverable, schema-documented, governed, and provenance-verified.

### Phase 3: Connect Layer Implementation

**Objective:** Enable cross-source linking and federated queries.

1. **Establish entity resolution.** Identify the common entities across your datasets (districts, organisations, products, etc.). Map them to canonical identifiers (LGD codes, HS codes, etc.) or create hashed common keys.

2. **Create JSON-LD contexts.** For each dataset, write a JSON-LD context document that maps every field to a URI from a recognized vocabulary. This is intellectually demanding work — it requires agreeing on what each field *means* in a shared vocabulary.

3. **Deploy federated query endpoint.** Set up a GraphQL (or SPARQL) endpoint that can route queries across multiple DIPs. This is the Virtual Data Layer.

4. **Populate the Mapping Registry.** If your datasets use domain-specific ontologies, register the mappings in the Cross-Domain Mapping Registry.

5. **Update Data Passports.** Add the Semantics section to each Data Passport.

**Deliverable:** Level 2 conformance. Datasets are linkable and queryable across institutions.

### Phase 4: Enable + Apply Layers

**Objective:** Make data AI-ready and deliver user-facing interfaces.

1. **Build the embedding pipeline.** Chunk text-heavy datasets, generate embeddings, and index them in a vector store. Publish Model Cards.

2. **Construct the knowledge graph.** Transform structured data into graph nodes and edges. Deploy a SPARQL or GraphQL endpoint.

3. **Implement the Metadata Bridge.** Connect the Knowledge Graph to the Vector Store so that agents can navigate from structured relationships to full-text content.

4. **Deploy MCP server.** Expose data products as MCP tools that AI agents can invoke.

5. **Build interfaces.** Develop dashboards, copilots, or automated workflows as defined in the Data Boarding Pass. Ensure all interfaces implement the citation/grounding schema.

6. **Implement audit logging.** Set up comprehensive logging for all data access events.

7. **Update Data Passports.** Add the AI Readiness section.

**Deliverable:** Level 3 conformance. Full AI-ready data infrastructure.

### Phase 5: Operationalise (Ongoing)

1. **Monitor data quality.** Implement automated quality checks against the Data Quality Scorecard.
2. **Manage embedding versioning.** When models are updated, follow the Vector Versioning Policy.
3. **Expand data sources.** Onboard additional DIPs using the same infrastructure.
4. **Measure outcomes.** Track the metrics defined in the Data Boarding Pass to demonstrate impact.

---

## Quick Start: Minimum Viable Deployment

For teams that want to see results quickly, here is the shortest path to a working Data Unlock deployment:

1. Take one dataset from one DIP
2. Standardise it in Parquet with a JSON Schema descriptor
3. Create a DCAT-AP catalogue entry
4. Generate embeddings and load them into a vector store
5. Deploy a basic MCP server that exposes the dataset as a queryable tool
6. Connect an LLM agent to the MCP server

This gives you a working "ask the data a question" capability in weeks rather than months. You can then progressively add governance (ODRL, VCs), linking (entity resolution, JSON-LD), and additional data sources.
