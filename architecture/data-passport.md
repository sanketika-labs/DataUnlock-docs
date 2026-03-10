# Data Passport

The Data Passport is the **machine-readable metadata envelope** that accompanies a data asset as it moves through the Data Unlock ecosystem. If the Data Boarding Pass describes a *deployment*, the Data Passport describes a *dataset* — its identity, quality, structure, provenance, and access rules.

## Concept

Think of a real passport. It doesn't contain the person — it contains verified claims *about* the person: identity, nationality, visa permissions, entry/exit stamps. Similarly, a Data Passport doesn't contain the data — it contains verified, machine-readable claims about the data: who produced it, what it contains, how good it is, who can access it, and what has happened to it.

Every data product that participates in a Data Unlock ecosystem must have a Data Passport. The passport travels with the data (or is resolvable via URI) so that any consuming system can make automated trust and fitness-for-use decisions without human intervention.

## Technical Manifestation: Specification

Like the Data Boarding Pass, the Data Passport is a **specification**, not software. It is defined as a JSON-LD envelope that combines several existing open standards into a single, coherent metadata structure.

The design choice to make this a specification (rather than a database or service) is critical:

1. **Portability.** The passport must travel with the data across institutional boundaries. A proprietary format would create lock-in.
2. **Verifiability.** Provenance claims are signed using W3C Verifiable Credentials, enabling cryptographic verification by any party.
3. **Composability.** The passport is an envelope that wraps existing standards (DCAT-AP for discovery, JSON Schema for structure, ODRL for policy, W3C VC for provenance). It doesn't replace them — it assembles them.
4. **Machine readability.** JSON-LD ensures that every field in the passport has an unambiguous URI-based meaning, enabling automated processing.

### Schema Definition

The Data Passport is structured as a JSON-LD document with the following sections:

```json
{
  "@context": [
    "https://www.w3.org/2018/credentials/v1",
    "https://dataunlock.org/contexts/data-passport/v1"
  ],
  "@type": "DataPassport",
  "@id": "urn:dataunlock:dp:example-dataset-v2",

  "identity": {
    "@type": "DataProductIdentity",
    "name": "District-level Agricultural Productivity, 2020-2025",
    "version": "2.1.0",
    "publisher": {
      "@type": "Organization",
      "name": "Ministry of Agriculture",
      "did": "did:web:agriculture.gov.example"
    },
    "issued": "2025-09-15T00:00:00Z",
    "modified": "2026-01-10T00:00:00Z",
    "language": ["en", "hi"],
    "spatialCoverage": "IN",
    "temporalCoverage": "2020-01-01/2025-12-31"
  },

  "discovery": {
    "@type": "dcat:Dataset",
    "description": "Seasonal crop yield, area sown, and production data at district granularity",
    "keyword": ["agriculture", "crop yield", "district", "India"],
    "theme": ["http://publications.europa.eu/resource/authority/data-theme/AGRI"],
    "catalogEndpoint": "https://catalogue.agriculture.gov.example/datasets/agri-prod-v2",
    "preview": {
      "format": "text/csv",
      "url": "https://data.agriculture.gov.example/preview/agri-prod-v2-sample.csv",
      "rowCount": 100
    }
  },

  "structure": {
    "@type": "DataProductStructure",
    "format": ["application/parquet", "text/csv"],
    "schema": {
      "$ref": "https://data.agriculture.gov.example/schemas/agri-prod-v2.json",
      "description": "JSON Schema with field types, constraints, and semantic annotations"
    },
    "frictionlessDescriptor": {
      "$ref": "https://data.agriculture.gov.example/datapackages/agri-prod-v2/datapackage.json"
    },
    "recordCount": 2450000,
    "columnCount": 24,
    "namingConvention": "snake_case"
  },

  "quality": {
    "@type": "DataQualityScorecard",
    "completeness": 0.94,
    "accuracy": 0.91,
    "timeliness": "monthly",
    "consistency": 0.88,
    "methodology": "https://data.agriculture.gov.example/quality/methodology-v1.pdf",
    "certifiedBy": {
      "@type": "Organization",
      "name": "Ministry of Agriculture - Data Quality Cell"
    },
    "certifiedOn": "2026-01-10T00:00:00Z"
  },

  "provenance": {
    "@type": "VerifiableCredential",
    "issuer": "did:web:agriculture.gov.example",
    "issuanceDate": "2026-01-10T00:00:00Z",
    "credentialSubject": {
      "id": "urn:dataunlock:dp:example-dataset-v2",
      "integrityHash": {
        "algorithm": "SHA-256",
        "value": "a1b2c3d4e5f6..."
      },
      "lineage": [
        {
          "source": "Primary data collection by state agriculture departments",
          "transformation": "Aggregated to district level, standardised to Parquet format",
          "agent": "Ministry of Agriculture - Statistics Division"
        }
      ]
    },
    "proof": {
      "type": "Ed25519Signature2020",
      "created": "2026-01-10T00:00:00Z",
      "verificationMethod": "did:web:agriculture.gov.example#key-1",
      "proofValue": "z58DAdF..."
    }
  },

  "policy": {
    "@type": "odrl:Policy",
    "@id": "urn:dataunlock:policy:agri-prod-v2",
    "permission": [
      {
        "target": "urn:dataunlock:dp:example-dataset-v2",
        "action": "use",
        "constraint": [
          {
            "leftOperand": "purpose",
            "operator": "isA",
            "rightOperand": "research"
          },
          {
            "leftOperand": "purpose",
            "operator": "isA",
            "rightOperand": "policymaking"
          }
        ]
      },
      {
        "target": "urn:dataunlock:dp:example-dataset-v2",
        "action": "distribute",
        "constraint": [
          {
            "leftOperand": "recipient",
            "operator": "isA",
            "rightOperand": "governmentAgency"
          }
        ],
        "duty": [
          {
            "action": "attribute",
            "target": "Ministry of Agriculture"
          }
        ]
      }
    ],
    "prohibition": [
      {
        "target": "urn:dataunlock:dp:example-dataset-v2",
        "action": "commercialize"
      }
    ]
  },

  "semantics": {
    "@type": "SemanticAnnotation",
    "context": "https://data.agriculture.gov.example/contexts/agri-prod-v2.jsonld",
    "ontologies": [
      "https://schema.org/",
      "http://aims.fao.org/aos/agrovoc"
    ],
    "entityResolution": {
      "districtIdentifier": {
        "type": "did:web",
        "registry": "https://lgdirectory.gov.in/"
      }
    }
  },

  "aiReadiness": {
    "@type": "AIReadinessMetadata",
    "embeddings": {
      "available": true,
      "modelCard": {
        "model": "text-embedding-3-small",
        "provider": "OpenAI",
        "dimensions": 1536,
        "version": "2024-01"
      },
      "vectorStoreEndpoint": "https://vectors.agriculture.gov.example/agri-prod-v2"
    },
    "knowledgeGraph": {
      "available": true,
      "format": "GraphML",
      "endpoint": "https://kg.agriculture.gov.example/sparql"
    },
    "mcpServer": {
      "available": true,
      "endpoint": "https://mcp.agriculture.gov.example/agri-prod-v2",
      "tools": ["query_crop_yield", "compare_districts", "get_time_series"]
    },
    "croissantMetadata": {
      "$ref": "https://data.agriculture.gov.example/croissant/agri-prod-v2.json"
    }
  }
}
```

## Sections Explained

### Identity
Who published this data, when, and what it covers. Uses DIDs for publisher identity, enabling cryptographic verification of the publisher's claims.

### Discovery
DCAT-AP compliant metadata that allows catalogue crawlers to automatically index this dataset. Includes a data preview (a sample of rows) so potential consumers can evaluate fitness before initiating a full transfer.

### Structure
Machine-readable schema definition using JSON Schema and Frictionless Data Package descriptors. Enables a consuming system to programmatically determine whether the data is structurally compatible with its needs.

### Quality
A standardised Data Quality Scorecard with metrics for completeness, accuracy, timeliness, and consistency. Certified by a named entity, enabling consumers to make trust decisions based on measured quality rather than reputation.

### Provenance
A W3C Verifiable Credential that cryptographically attests to the dataset's origin, transformation history, and integrity. The hash allows any consumer to verify that the data has not been tampered with since the credential was issued.

### Policy
ODRL 2.2 policy expressions that define who can access the data, for what purposes, under what conditions, and with what obligations. Policies travel with the data — they are not locked inside the source system's access control layer.

### Semantics
JSON-LD context documents and ontology references that give every field in the dataset an unambiguous, machine-interpretable meaning. Includes entity resolution metadata specifying which identifier system is used for cross-dataset linking.

### AI Readiness
Metadata about the AI-ready representations of this dataset: whether embeddings are available (and from which model), whether a knowledge graph representation exists, whether an MCP server exposes it, and whether Croissant metadata is available for ML pipeline consumption.

## Lifecycle

A Data Passport is not a static document. It evolves with the dataset:

1. **Created** when a DIP first publishes a data product through the Prepare layer.
2. **Updated** each time the dataset is refreshed, re-certified, or its policies change. Each update produces a new version with a new provenance credential.
3. **Extended** as the dataset progresses through the value chain. Initially, only the Identity, Discovery, Structure, Quality, Provenance, and Policy sections are populated. As the Connect layer is implemented, the Semantics section is added. As the Enable layer is implemented, the AI Readiness section is added.
4. **Archived** when the dataset is deprecated or superseded.

## Relationship to the Data Boarding Pass

The Data Boarding Pass and Data Passport are complementary:

| | Data Boarding Pass | Data Passport |
|---|---|---|
| **Describes** | A deployment (use case) | A dataset (data product) |
| **Scope** | Multiple data sources working together | A single data source |
| **Audience** | Project managers, Nodal Agencies | Data engineers, consuming systems |
| **References** | Contains URIs to Data Passports of its data sources | Exists independently; may be referenced by multiple Boarding Passes |
| **Lifecycle** | Created when a use case is initiated | Created when a dataset is first published |

A single Data Passport may be referenced by many Data Boarding Passes (the same dataset serves multiple use cases). A single Data Boarding Pass always references one or more Data Passports.
