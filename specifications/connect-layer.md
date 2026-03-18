# Connect: Semantic Intelligence

**Goal:** Resolve entities and align meanings across disparate data sources without centralising the data.

The Connect layer creates the "mesh" that transforms individual trusted data products into a federated knowledge layer. After this stage, a query can span multiple datasets from different institutions as if they were a single source.

---

## Entity Resolution

### Specification

Adopters **must** implement entity resolution to match entities (places, organisations, products, persons) across datasets without exposing PII.

| Attribute | Requirement |
|---|---|
| **Identifier standard** | [W3C Decentralised Identifiers (DIDs)](https://www.w3.org/TR/did-core/) for institutional entities; SHA-256 hashed common keys for personal/sensitive entities |
| **Resolution method** | Deterministic matching on shared identifiers (preferred) or probabilistic matching with confidence scores |
| **PII protection** | Raw PII must never be transmitted for matching. Use one-way hashed keys (e.g., SHA-256 of a tax ID) or privacy-preserving record linkage techniques |
| **Identifier registry** | A shared registry of canonical entity identifiers must be maintained by the Nodal Agency or delegated to a trusted third party |

### Recommended Identifier Systems

| Entity type | Recommended identifier | Example |
|---|---|---|
| Countries | ISO 3166-1 | `IN`, `AU`, `US` |
| Sub-national regions | National registry (e.g., LGD in India) | `LGD:632` (Bangalore Urban) |
| Products (trade) | HS Code (WCO) | `0901.11` (Coffee, not roasted) |
| Enterprises | National registration ID (e.g., Udyam, GSTIN) | `UDYAM-KA-26-0012345` |
| Statistical indicators | SDMX concept identifiers | `INDICATOR:GDP_CURRENT_PRICES` |
| Persons (when required) | Hashed national ID | `SHA256(aadhaar_number)` |

### Example: Entity Resolution Mapping

```json
{
  "@context": "https://dataunlock.org/contexts/entity-resolution/v1",
  "@type": "EntityResolutionMapping",
  "sourceA": {
    "dataset": "urn:dataunlock:dp:moci-trade-stats-v2",
    "field": "district_name",
    "sampleValue": "Bangalore Urban"
  },
  "sourceB": {
    "dataset": "urn:dataunlock:dp:mospi-economic-indicators-v3",
    "field": "district_code",
    "sampleValue": "632"
  },
  "resolution": {
    "canonicalIdentifier": "lgd:district:632",
    "registry": "https://lgdirectory.gov.in/",
    "confidence": 1.0,
    "method": "deterministic"
  }
}
```

### Crosswalk API Endpoint

Static entity resolution mapping documents (above) are essential for documentation, but automated pipelines and AI agents need a **runtime crosswalk service** to resolve identifiers on the fly. A DIP or Nodal Agency that publishes multiple datasets with different internal identifier systems **should** expose a crosswalk API endpoint.

| Attribute | Requirement |
|---|---|
| **Endpoint** | REST API returning JSON-LD, queryable by entity type, source dataset, and identifier value |
| **Supported entity types** | At minimum: geographic entities (states, districts), time periods, and classification codes (industry, commodity) |
| **Response** | Returns the canonical identifier and all known dataset-specific equivalents |
| **Bulk mode** | The endpoint should support batch resolution (multiple identifiers in a single request) for pipeline use cases |

#### Example: Crosswalk Query

**Request:** `GET /crosswalk?entity_type=state&source=plfs&value=25`

**Response:**

```json
{
  "@context": "https://dataunlock.org/contexts/entity-resolution/v1",
  "@type": "CrosswalkResult",
  "entity_type": "state",
  "canonical": {
    "identifier": "lgd:state:33",
    "label": "Tamil Nadu",
    "registry": "https://lgdirectory.gov.in/"
  },
  "equivalents": [
    { "dataset": "plfs", "field": "state_code", "value": "25" },
    { "dataset": "cpi", "field": "state_code", "value": "33" },
    { "dataset": "asi", "field": "state_code", "value": "22" },
    { "dataset": "census_2011", "field": "state_code", "value": "33" },
    { "dataset": "iso_3166_2", "field": "code", "value": "IN-TN" }
  ]
}
```

#### Example: Temporal Crosswalk Query

**Request:** `GET /crosswalk?entity_type=time_period&source=plfs&value=2023-24`

**Response:**

```json
{
  "@type": "CrosswalkResult",
  "entity_type": "time_period",
  "canonical": {
    "financial_year": "2023-24",
    "calendar_year_start": 2023,
    "calendar_year_end": 2024,
    "format": "financial_year_apr_mar"
  },
  "equivalents": [
    { "dataset": "plfs", "field": "year", "value": "2023-24", "format": "financial_year" },
    { "dataset": "cpi", "field": "year", "value": "2024", "note": "Maps to calendar year of the financial year's second half" },
    { "dataset": "nas", "field": "year", "value": "2023-24", "format": "financial_year" }
  ]
}
```

This crosswalk endpoint is not a replacement for adopting standard identifiers (which remains the long-term goal) — it is a bridge that makes cross-dataset analysis viable while the ecosystem transitions to consistent identifiers.

---

## Semantic Alignment

### Specification

Every field in a data product **must** be mapped to a URI from a recognised vocabulary, ensuring unambiguous machine-interpretable meaning.

| Attribute | Requirement |
|---|---|
| **Mapping format** | [JSON-LD 1.1](https://www.w3.org/TR/json-ld11/) context documents |
| **Primary vocabulary** | [Schema.org](https://schema.org/) for general-purpose concepts |
| **Domain vocabularies** | AGROVOC (agriculture), SDMX (statistics), HS Nomenclature (trade), SNOMED (health), etc. |
| **Mapping granularity** | Every field must link to a URI. Fields that do not match any existing vocabulary term must be registered in a domain-specific extension vocabulary. |
| **Conflict resolution** | When two adopters use different ontologies for the same domain, the Cross-Domain Mapping Registry mediates |

### Why JSON-LD?

JSON-LD allows semantic context to be layered on top of existing JSON data without changing its structure. A dataset can remain a plain JSON or CSV file while its JSON-LD context document provides the semantic "translation layer" that tells consuming systems what each field means.

### Example: JSON-LD Context

```json
{
  "@context": {
    "@vocab": "https://schema.org/",
    "du": "https://dataunlock.org/vocab/",
    "agrovoc": "http://aims.fao.org/aos/agrovoc/",

    "district_code": {
      "@id": "du:districtIdentifier",
      "@type": "xsd:string",
      "sameAs": "https://lgdirectory.gov.in/vocab/districtCode"
    },
    "crop_name": {
      "@id": "agrovoc:c_1972",
      "@type": "xsd:string",
      "description": "Standardised crop name from AGROVOC"
    },
    "yield_kg_per_hectare": {
      "@id": "du:cropYield",
      "@type": "xsd:decimal",
      "unitCode": "KGM/HAR",
      "description": "Crop yield measured in kilograms per hectare"
    },
    "season": {
      "@id": "du:agriculturalSeason",
      "@type": "xsd:string"
    }
  }
}
```

### Cross-Domain Mapping Registry

When DIP A and DIP B use different ontologies for the same domain (e.g., two different agricultural vocabularies), the **Cross-Domain Mapping Registry** provides a mediation layer. This is a shared service operated by (or delegated by) the Nodal Agency.

The registry stores `owl:equivalentClass` and `owl:equivalentProperty` mappings:

```turtle
du:cropYield owl:equivalentProperty fao:yieldPerHectare .
mospi:district_id owl:sameAs lgd:districtCode .
```

---

## Federated Query

### Specification

Adopters **must** support a standard query interface that allows "join" operations across distributed sources without requiring data centralisation.

| Attribute | Requirement |
|---|---|
| **Query language** | [GraphQL](https://graphql.org/) (primary) or [SPARQL 1.1](https://www.w3.org/TR/sparql11-query/) (for RDF-native stores) |
| **Federation** | Queries must be federatable via a Virtual Data Layer (VDL) that routes sub-queries to the appropriate DIP |
| **Schema stitching** | The VDL must support GraphQL schema stitching or SPARQL SERVICE federation to combine results from multiple endpoints |
| **Authentication** | Federated queries must carry the requester's identity and consent tokens; each DIP validates access policy independently |

### Architecture: Virtual Data Layer

```
        ┌─────────────────────────┐
        │    Consuming System     │
        │ (sends a single query)  │
        └───────────┬─────────────┘
                    │
                    ▼
        ┌─────────────────────────┐
        │   Virtual Data Layer    │
        │ (query router/planner)  │
        └──┬──────────┬──────────┬┘
           │          │          │
           ▼          ▼          ▼
     ┌──────────┐ ┌──────────┐ ┌──────────┐
     │  DIP A   │ │  DIP B   │ │  DIP C   │
     │ (GraphQL)│ │ (SPARQL) │ │ (GraphQL)│
     └──────────┘ └──────────┘ └──────────┘
```

The VDL decomposes a federated query into sub-queries, routes each to the appropriate DIP, and assembles the results. Each DIP validates the requester's permissions independently against its ODRL policies.

---

## ML Metadata: Croissant

### Specification

Datasets intended for ML pipeline consumption **should** include [Croissant](https://mlcommons.org/croissant/) metadata descriptors.

| Attribute | Requirement |
|---|---|
| **Standard** | Croissant 1.0 (MLCommons) |
| **Purpose** | Describes dataset contents (columns, types, bias information) so ML frameworks can auto-load the data |
| **Framework support** | Enables one-line loading into PyTorch, TensorFlow, and HuggingFace Datasets |

Croissant acts as a "nutritional label for data" — it tells an ML pipeline exactly what is in the dataset so it doesn't have to guess. This is particularly valuable for bridging the gap between raw statistical data and modern AI pipelines.

---

## Open Items

### 1. Vocabulary Collision

When two adopters use different ontologies for the same domain, the Connect layer's semantic alignment will fail unless a mediation mechanism exists.

**Current solution:** Cross-Domain Mapping Registry (described above).
**Status:** Specification for the registry's API and data model is under development.

### 2. Consent Propagation

There is no standard for how a consent token generated at DIP A's portal is technically validated by DIP B's data server during a real-time federated query.

**Proposed solution:** A Federated Consent Token format, likely based on JWT with ODRL claims, that can be validated independently by each participant in a federated query chain.
**Status:** Under active design. This is a critical gap that must be resolved before production federated queries are deployed.
