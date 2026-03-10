# What is Data Unlock?

Data Unlock is a trustworthy mechanism for exchanging data that creates value for all participants through built-in incentives. It is designed as a **Digital Public Infrastructure (DPI)** — a shared, open framework that any government or institution can adopt to make their data ecosystems interoperable and AI-ready.

## The One-Line Definition

> Data Unlock turns fragmented, siloed government data into trusted, linked, machine-readable intelligence — without centralising it.

## What it does

Data Unlock addresses four fundamental gaps that exist in most government data ecosystems today:

| Gap | Description | How Data Unlock addresses it |
|---|---|---|
| **Discovery gap** | Data exists, but is hard to find. Datasets are buried in PDFs, spreadsheets, and departmental databases with no unified catalogue. | Standardised metadata catalogues using DCAT-AP, with machine-readable descriptors and data previews. |
| **Structuring gap** | Data lacks context and is not structured for use. Column names are ambiguous, formats vary, and there is no schema documentation. | JSON Schema descriptors, Frictionless Data Packages, and standardised naming conventions (snake_case, Parquet/Avro). |
| **Silo gap** | Datasets don't link across actors and systems. A "District" in one dataset has no machine-readable connection to the same district in another. | Entity resolution via Decentralised Identifiers (DIDs), semantic alignment via JSON-LD and Schema.org ontologies, and federated query via GraphQL/SPARQL. |
| **Machine gap** | Data is human-readable, not AI-ready. You can read a PDF, but an LLM cannot reliably extract structured facts from it. | Embeddings with model cards, knowledge graphs (GraphML/JSON-LD), vector stores, and MCP server interfaces for AI agents. |

## What Data Unlock is NOT

It is important to be precise about what this framework does and does not prescribe:

- **Not a data lake.** Data Unlock does not require centralising data into a single repository. Data remains with its custodian institution.
- **Not a platform.** There is no single software product called "Data Unlock." It is an architecture — a set of specifications, protocols, and roles that can be implemented using various technology stacks.
- **Not a mandate.** Adoption is incremental. An institution can begin by implementing just the Prepare layer (cataloguing and standardisation) and progressively adopt Connect, Enable, and Apply capabilities.

## The DPI Framing

Data Unlock follows the DPI model that has been successfully applied in India (Aadhaar, UPI, DigiLocker) and is increasingly adopted globally:

- **Open specifications** that any vendor can implement
- **Federated architecture** where each participant runs their own infrastructure
- **Interoperability by design** through shared protocols rather than shared platforms
- **Population-scale** ambition with incremental adoption paths

The framework produces two primary artefacts that implementers work with: the [Data Boarding Pass](../architecture/data-boarding-pass.md) (a deployment manifest for a specific use case) and the [Data Passport](../architecture/data-passport.md) (machine-readable metadata that travels with data assets).
