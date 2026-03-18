# Prepare: Foundation & Trust

**Goal:** Turn fragmented data into standardised, high-quality, governed data products with verifiable provenance.

This is the foundational layer. No data enters the Data Unlock ecosystem without passing through the Prepare stage. It ensures that every data product is discoverable, understandable, governed, and trustworthy before any enrichment or AI processing occurs.

---

## Discover: Metadata Catalogues

### Specification

Every Data Information Provider (DIP) **must** expose a DCAT-AP compliant metadata catalogue endpoint.

| Attribute | Requirement |
|---|---|
| **Standard** | [DCAT-AP 3.0](https://semiceu.github.io/DCAT-AP/releases/3.0.0/) (Data Catalogue Vocabulary - Application Profile) |
| **Transport** | HTTPS REST endpoint returning JSON-LD or Turtle (RDF) |
| **Minimum fields** | `dct:title`, `dct:description`, `dct:publisher`, `dcat:distribution`, `dct:temporal`, `dct:spatial`, `dcat:keyword`, `dcat:theme` |
| **Catalogue harvesting** | Endpoint must support [OAI-PMH](http://www.openarchives.org/OAI/openarchivesprotocol.html) or [CSW](https://www.ogc.org/standard/cat/) for automated cross-adopter indexing |
| **Data previews** | Each dataset entry should include a `dcat:distribution` with a preview resource (first 100 rows in CSV) |

### Why DCAT-AP?

DCAT-AP enables automated cross-institution catalogue crawling. When DIP A exposes a DCAT-AP endpoint, DIP B's catalogue can automatically discover and index A's datasets without custom integration. This is the same standard used by the European Open Data Portal and hundreds of national data portals.

### API Discovery via DCAT Data Services

To enable standardised discovery of API endpoints across the Data Unlock ecosystem, each DCAT-AP catalogue entry **must** include `dcat:DataService` references that map data products to their available access interfaces. This eliminates the need for a separate service registry — the catalogue itself serves as the "DNS for data APIs."

| Attribute | Requirement |
|---|---|
| **`dcat:DataService`** | Each access interface (REST API, MCP server, SPARQL endpoint, bulk download) must be described as a `dcat:DataService` entry |
| **`dcat:endpointURL`** | The root URL of the running service instance |
| **`dcat:endpointDescription`** | A URL pointing to the machine-readable interface description (OpenAPI spec, MCP manifest, SPARQL service description, etc.) |
| **`dct:conformsTo`** | The protocol or standard the service implements (e.g., OpenAPI 3.1, MCP, SPARQL 1.1, S3) |
| **`dcat:servesDataset`** | Back-reference to the `dcat:Dataset` URI(s) served by this endpoint |

This approach leverages the existing DCAT-AP harvesting mechanism — when DIP A's catalogue is crawled, the consuming system automatically discovers not just *what data exists* but *how to access it programmatically*. No additional discovery protocol is required.

### Example Endpoint Response

```json
{
  "@context": "https://www.w3.org/ns/dcat#",
  "@type": "dcat:Dataset",
  "dct:title": "District-level Crop Yield Statistics",
  "dct:description": "Seasonal crop production data at district granularity",
  "dct:publisher": {
    "@type": "foaf:Organization",
    "foaf:name": "Ministry of Agriculture"
  },
  "dct:temporal": {
    "dcat:startDate": "2020-01-01",
    "dcat:endDate": "2025-12-31"
  },
  "dct:spatial": "http://publications.europa.eu/resource/authority/country/IND",
  "dcat:keyword": ["agriculture", "crop yield", "district"],
  "dcat:distribution": [
    {
      "@type": "dcat:Distribution",
      "dcat:accessURL": "https://data.agri.gov.example/api/v1/crop-yield",
      "dct:format": "application/parquet"
    },
    {
      "@type": "dcat:Distribution",
      "dcat:accessURL": "https://data.agri.gov.example/preview/crop-yield-sample.csv",
      "dct:format": "text/csv",
      "dct:description": "Preview (first 100 rows)"
    }
  ],
  "dcat:hasService": [
    {
      "@type": "dcat:DataService",
      "dct:title": "Crop Yield REST API",
      "dcat:endpointURL": "https://data.agri.gov.example/api/v2/crop-yield",
      "dcat:endpointDescription": "https://data.agri.gov.example/api/v2/openapi.json",
      "dct:conformsTo": "https://spec.openapis.org/oas/v3.1.0",
      "dcat:servesDataset": "urn:dataunlock:dp:crop-yield-v2"
    },
    {
      "@type": "dcat:DataService",
      "dct:title": "Crop Yield MCP Server",
      "dcat:endpointURL": "https://data.agri.gov.example/mcp",
      "dcat:endpointDescription": "https://data.agri.gov.example/mcp/manifest.json",
      "dct:conformsTo": "https://modelcontextprotocol.io/specification",
      "dcat:servesDataset": "urn:dataunlock:dp:crop-yield-v2"
    },
    {
      "@type": "dcat:DataService",
      "dct:title": "Agriculture Knowledge Graph SPARQL Endpoint",
      "dcat:endpointURL": "https://data.agri.gov.example/sparql",
      "dcat:endpointDescription": "https://data.agri.gov.example/sparql/service-description",
      "dct:conformsTo": "https://www.w3.org/TR/sparql11-protocol/",
      "dcat:servesDataset": "urn:dataunlock:dp:crop-yield-v2"
    },
    {
      "@type": "dcat:DataService",
      "dct:title": "Crop Yield Bulk Download",
      "dcat:endpointURL": "https://data.agri.gov.example/bulk/crop-yield/",
      "dcat:endpointDescription": "https://data.agri.gov.example/bulk/crop-yield/manifest.json",
      "dct:conformsTo": "https://specs.frictionlessdata.io/data-package/",
      "dcat:servesDataset": "urn:dataunlock:dp:crop-yield-v2"
    }
  ]
}
```

---

## Understand: Schema Descriptors

### Specification

Every data product **must** include a machine-readable schema descriptor that enables consuming systems to programmatically evaluate fitness for use.

| Attribute | Requirement |
|---|---|
| **Schema format** | [JSON Schema (2020-12)](https://json-schema.org/specification) |
| **Data packaging** | [Frictionless Data Package](https://specs.frictionlessdata.io/data-package/) descriptor (`datapackage.json`) |
| **Required metadata** | Data types, constraints (min/max, regex), temporal coverage, update frequency, units of measurement |
| **Naming conventions** | All field names in `snake_case` |

### Why JSON Schema + Frictionless?

JSON Schema provides the structural contract (types, constraints), while Frictionless Data Package provides the contextual wrapper (licensing, sources, contributors). Together, they enable a machine from consuming system B to pre-determine whether data from DIP A is fit for use before initiating a transfer.

### Example: datapackage.json

```json
{
  "name": "district-crop-yield",
  "title": "District-level Crop Yield Statistics",
  "version": "2.1.0",
  "licenses": [
    {
      "name": "OGL-India-2.0",
      "path": "https://data.gov.in/government-open-data-license-india"
    }
  ],
  "resources": [
    {
      "name": "crop_yield",
      "path": "crop_yield.parquet",
      "format": "parquet",
      "mediatype": "application/parquet",
      "schema": {
        "fields": [
          {
            "name": "district_code",
            "type": "string",
            "description": "LGD district code",
            "constraints": { "required": true, "pattern": "^[0-9]{3,4}$" }
          },
          {
            "name": "crop_name",
            "type": "string",
            "description": "Standardised crop name from ICAR vocabulary"
          },
          {
            "name": "season",
            "type": "string",
            "constraints": { "enum": ["kharif", "rabi", "zaid"] }
          },
          {
            "name": "yield_kg_per_hectare",
            "type": "number",
            "description": "Crop yield in kilograms per hectare",
            "constraints": { "minimum": 0 }
          },
          {
            "name": "year",
            "type": "integer",
            "constraints": { "minimum": 2000, "maximum": 2030 }
          }
        ],
        "primaryKey": ["district_code", "crop_name", "season", "year"]
      }
    }
  ]
}
```

---

## Curate: Normalised Data Views

### Specification

All data products exposed through Data Unlock **must** be available in open, vendor-neutral formats with standardised conventions.

| Attribute | Requirement |
|---|---|
| **File formats** | [Apache Parquet](https://parquet.apache.org/) (primary) or [Apache Avro](https://avro.apache.org/) (for streaming). CSV as a secondary distribution. |
| **Naming** | All column names in `snake_case`. No spaces, no special characters except underscores. |
| **Encoding** | UTF-8 |
| **Date/time** | ISO 8601 (`YYYY-MM-DD`, `YYYY-MM-DDTHH:MM:SSZ`) |
| **Null handling** | Explicit nulls (not empty strings, not "N/A", not "-") |
| **Geographic identifiers** | ISO 3166 for countries, LGD codes (or equivalent national registry) for sub-national units |
| **Currency** | ISO 4217 codes |

### Why Parquet/Avro?

These formats prevent vendor lock-in and ensure that cleaned data remains usable across different compute environments (Spark, Pandas, DuckDB, BigQuery, etc.). They also support columnar compression, schema evolution, and type enforcement — critical for data quality at scale.

---

## Authorise: Access Policies

### Specification

All access permissions **must** be encoded as [ODRL 2.2](https://www.w3.org/TR/odrl-model/) policy expressions. Policies travel with the data (embedded in or referenced from the Data Passport), not locked inside the source system.

| Attribute | Requirement |
|---|---|
| **Standard** | ODRL 2.2 (Open Digital Rights Language) |
| **Encoding** | JSON-LD |
| **Required elements** | At least one `permission` with `target`, `action`, and `constraint` |
| **Purpose binding** | Every permission **must** include a `purpose` constraint |
| **Enforcement point** | Policies are validated at the Access layer (Enable stage), not just at the source |

### Why ODRL?

ODRL enables "policy portability" — the rules governing data stay attached to the data as it moves across the ecosystem. When a dataset flows from DIP A to Data Unlocker B to DIU C, the access policy travels with it. DIU C's system can programmatically verify whether its intended use is permitted, without calling back to DIP A.

### Supplementary: Rego Policies

For complex, programmatic policy evaluation at runtime (e.g., "allow access only if the requester's consent token is valid and their role matches 'researcher'"), implementations **should** supplement ODRL with [Open Policy Agent (OPA)](https://www.openpolicyagent.org/) using Rego policies. ODRL defines the policy; Rego evaluates it at the enforcement point.

### Example: ODRL Policy

```json
{
  "@context": "http://www.w3.org/ns/odrl.jsonld",
  "@type": "odrl:Offer",
  "uid": "urn:dataunlock:policy:crop-yield-v2",
  "profile": "http://dataunlock.org/odrl-profile/v1",
  "permission": [
    {
      "target": "urn:dataunlock:dp:crop-yield-v2",
      "assigner": "did:web:agriculture.gov.example",
      "action": "use",
      "constraint": [
        {
          "leftOperand": "purpose",
          "operator": "isAnyOf",
          "rightOperand": ["research", "policymaking", "publicGood"]
        }
      ],
      "duty": [
        {
          "action": "attribute",
          "informedParty": "Ministry of Agriculture"
        }
      ]
    }
  ],
  "prohibition": [
    {
      "target": "urn:dataunlock:dp:crop-yield-v2",
      "action": "commercialize"
    }
  ]
}
```

---

## Assure: Provenance & Verifiability

### Specification

Every dataset version **must** be signed with a cryptographic hash and accompanied by a [W3C Verifiable Credential](https://www.w3.org/TR/vc-data-model-2.0/) attesting to its provenance.

| Attribute | Requirement |
|---|---|
| **Standard** | W3C Verifiable Credentials Data Model 2.0 |
| **Integrity** | SHA-256 hash of the dataset file(s) |
| **Signature** | Ed25519Signature2020 or JsonWebSignature2020 |
| **Issuer identity** | DID (Decentralised Identifier) of the publishing institution |
| **Lineage** | Credential must include a `lineage` claim describing data sources and transformations |

### Why W3C Verifiable Credentials?

VCs establish an immutable, cryptographically verifiable audit trail. When Data Unlocker B receives a dataset from DIP A, it can mathematically verify that the data has not been tampered with since A signed it. This is essential for building trust in a federated ecosystem where data crosses institutional boundaries.

### Example: Provenance Credential

```json
{
  "@context": ["https://www.w3.org/2018/credentials/v1"],
  "type": ["VerifiableCredential", "DataProvenanceCredential"],
  "issuer": "did:web:agriculture.gov.example",
  "issuanceDate": "2026-01-10T00:00:00Z",
  "credentialSubject": {
    "id": "urn:dataunlock:dp:crop-yield-v2",
    "name": "District-level Crop Yield Statistics v2.1.0",
    "integrityHash": {
      "algorithm": "SHA-256",
      "value": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"
    },
    "lineage": [
      {
        "source": "State agriculture department field surveys",
        "transformation": "Aggregation from village-level to district-level; unit normalisation to kg/hectare",
        "agent": "Ministry of Agriculture - Statistics Division",
        "timestamp": "2026-01-08T00:00:00Z"
      }
    ]
  },
  "proof": {
    "type": "Ed25519Signature2020",
    "created": "2026-01-10T00:00:00Z",
    "verificationMethod": "did:web:agriculture.gov.example#key-1",
    "proofPurpose": "assertionMethod",
    "proofValue": "z58DAdFfa9SkqZMVPxAQpic7ndTn..."
  }
}
```

---

## Data Quality Scorecard

Without a shared definition of quality, consuming systems may ingest data that is technically "standardised" but factually inaccurate. The Data Quality Scorecard defines a standard set of metrics that every DIP **must** publish for each data product as part of its Data Passport.

The scorecard is organised into three tiers: **Core Metrics** (fundamental quality dimensions applicable to any dataset), **Structural Integrity Metrics** (measuring whether the data's technical representation is correct and consistent), and **Operational Metrics** (measuring the reliability of the data as a living, maintained product).

### Core Metrics

| Metric | Description | Measurement | Example |
|---|---|---|---|
| **Completeness** | Percentage of non-null values across required fields | 0.0 – 1.0 | A dataset with 94% non-null values across required fields scores 0.94 |
| **Accuracy** | Percentage of values that pass domain-specific validation rules | 0.0 – 1.0 | If 3% of district codes fail to resolve against the LGD master list, accuracy = 0.97 |
| **Uniqueness** | Percentage of records with no unintended duplicates on the declared primary key | 0.0 – 1.0 | If a dataset returns the same record twice for a given year + state + indicator combination, uniqueness < 1.0 |
| **Validity** | Percentage of values that conform to the declared schema constraints (type, range, enum, pattern) | 0.0 – 1.0 | If a numeric field contains non-parseable string values (e.g., "N/A" instead of null), validity < 1.0 |

### Structural Integrity Metrics

These metrics address data representation issues that create silent failures in consuming systems — issues often invisible during casual inspection but critical for automated pipelines.

| Metric | Description | Measurement | Example |
|---|---|---|---|
| **Type Fidelity** | Percentage of fields returned in their declared data type (not as strings or mismatched types) | 0.0 – 1.0 | If an API returns numeric values as strings (e.g., "3.20" instead of 3.20), type fidelity < 1.0. Consuming systems must then parse strings, handle edge cases, and risk precision loss |
| **Null Semantics** | Whether the dataset documents null patterns and distinguishes between "not collected", "not applicable", "zero", and "error" | Boolean + documentation | A dataset where missing values silently omit the record (rather than returning null) scores poorly — the consumer cannot distinguish "data not collected" from "query error" |
| **Unit Standardisation** | Whether all value fields have a documented unit of measurement from a controlled vocabulary | Boolean per field | If one dataset returns "%" as the unit, another returns "₹ crore", and a third returns values with no unit metadata, unit standardisation fails |
| **Precision Appropriateness** | Whether numeric values use appropriate decimal precision for the domain | Boolean | GDP values returned with 15-digit decimal precision (e.g., "32411405.953773014") imply false accuracy when the underlying data is measured in crores |
| **Identifier Consistency** | Whether entity identifiers (geographic codes, classification codes, temporal formats) are consistent across all datasets published by the same DIP | 0.0 – 1.0 | If state codes use one numbering system in dataset A and an entirely different system in dataset B, cross-dataset analysis requires a manual mapping table. This is a critical failure for automated pipelines |
| **Naming Convention Compliance** | Whether field names, parameter names, and filter codes follow documented naming conventions, use consistent terminology across datasets, and are semantically accurate (i.e., the name correctly describes what the parameter controls). A parameter whose name implies one function but actually controls something else scores poorly even if consistently named. | 0.0 – 1.0 | If time period is called `year` in one dataset (with format YYYY-YY), `year` in another (with format YYYY), and `quarterly_code` in a third, naming convention compliance is low. Similarly, if a parameter named `frequency_code` with values "annual/quarterly/monthly" actually selects different *indicator sets* rather than controlling time granularity, it is semantically misleading and scores poorly. |

### Operational Metrics

These metrics measure whether the data product is reliably maintained and operationally fit for automated consumption.

| Metric | Description | Measurement | Example |
|---|---|---|---|
| **Timeliness** | The lag between the reference period end date and actual publication/availability | Duration (ISO 8601) | If CPI data for January 2026 is published on 12 February 2026, timeliness = P12D. Shorter is better |
| **Update Adherence** | Whether the dataset is updated according to its declared schedule | 0.0 – 1.0 | If a monthly dataset has missed 2 of the last 12 updates, adherence = 0.83 |
| **Release Calendar** | Whether a machine-readable publication schedule exists, documenting when the next update is expected | Boolean | Automated pipelines need to know when to re-fetch data. Without a release calendar, consumers must poll blindly |
| **Revision Transparency** | Whether the dataset identifies revision vintages and provides a programmatic way to retrieve the latest/most authoritative version | Ordinal: none / partial / full | If a dataset includes multiple revision stages (advance estimate, revised, final) but provides no field indicating which is most current, consuming systems must hardcode priority logic |
| **Error Diagnostics** | Whether API error responses distinguish between invalid requests, not-yet-published data, empty result sets, and server errors | Ordinal: none / basic / informative | If every failed query returns the same generic empty response regardless of cause, consumers cannot distinguish between a malformed request and data that does not exist |
| **Pagination Transparency** | Whether the API documents its result size limits and clearly signals when results are truncated | Boolean | If an API silently returns only the first 10 records of a 210-record result set, consuming systems may unknowingly operate on incomplete data |
| **Methodology Reference** | Whether the data product links to its underlying collection methodology, survey design, or technical documentation | Boolean | Users unfamiliar with domain-specific terminology cannot correctly interpret data without access to the methodology. Changes in methodology across time periods affect comparability |
| **Data Dictionary** | Whether every field/indicator has a formal definition, not just a label | Boolean | A field labelled "LFPR (Labour Force Participation Rate, in per cent)" provides a label but not a definition. A data dictionary would explain the methodology, reference population, and computation |

### Scorecard Schema

Each Data Passport **must** include a quality section conforming to this structure:

```json
{
  "quality": {
    "scorecardVersion": "1.0.0",
    "assessmentDate": "2026-01-15",
    "core": {
      "completeness": 0.94,
      "accuracy": 0.97,
      "uniqueness": 1.0,
      "validity": 0.91
    },
    "structuralIntegrity": {
      "typeFidelity": 0.85,
      "nullSemantics": true,
      "nullDocumentation": "Null values indicate data not collected. Zero values are explicit. Missing records indicate the geographic unit was not surveyed in that period.",
      "unitStandardisation": true,
      "precisionAppropriateness": true,
      "identifierConsistency": 1.0,
      "namingConventionCompliance": 0.95
    },
    "operational": {
      "timeliness": "P14D",
      "updateAdherence": 0.92,
      "releaseCalendar": true,
      "nextExpectedUpdate": "2026-03-15",
      "revisionTransparency": "full",
      "errorDiagnostics": "informative",
      "paginationTransparency": true,
      "methodologyReference": "https://mospi.gov.in/publication/plfs-methodology-2024",
      "dataDictionary": true
    }
  }
}
```

### Scoring Guidance

For each metric, DIPs should self-assess and publish scores. The Nodal Agency may also conduct independent verification. A dataset's overall quality posture is determined by the lowest-scoring tier:

| Tier | Target | Interpretation |
|---|---|---|
| **Core Metrics** | All ≥ 0.90 | The data itself is fundamentally sound |
| **Structural Integrity** | All boolean = true, all numeric ≥ 0.90 | The data's technical representation is reliable for automated consumption |
| **Operational** | All boolean = true, adherence ≥ 0.85 | The data product is reliably maintained and well-documented |

A dataset that scores well on Core Metrics but poorly on Structural Integrity (e.g., returns all values as strings, uses inconsistent identifiers) may be usable by a human analyst but will cause systematic failures in automated pipelines and AI agents.
