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

## Open Item: Data Quality Scorecard

The current specification identifies a need for a standardised **Data Quality Scorecard** schema. Without a shared definition of quality, consuming systems may ingest data that is technically "standardised" but factually inaccurate.

The scorecard should define metrics for:

| Metric | Description | Measurement |
|---|---|---|
| **Completeness** | Percentage of non-null values across required fields | 0.0 – 1.0 |
| **Accuracy** | Percentage of values that pass domain-specific validation rules | 0.0 – 1.0 |
| **Timeliness** | Update frequency and lag between collection and publication | Duration (ISO 8601) |
| **Consistency** | Percentage of values consistent across related fields and across time | 0.0 – 1.0 |
| **Uniqueness** | Percentage of records with no unintended duplicates | 0.0 – 1.0 |

> **Status:** This scorecard schema is under development. Implementations should define their own quality metrics in the interim, using the structure above as a guide. A standardised JSON Schema for the scorecard will be published in a future version of this specification.
