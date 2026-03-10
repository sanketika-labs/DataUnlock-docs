# Data Boarding Pass

The Data Boarding Pass is the **deployment manifest** for a specific Data Unlock use case. If Data Unlock is the architecture, the Data Boarding Pass is the "ticket" that describes a particular journey through it — who is travelling (data sources), where they are going (purpose), who benefits (users), and what vehicle they are taking (interfaces).

## Concept

The metaphor is deliberate. Just as an airline boarding pass encodes everything needed to get a specific passenger on a specific flight, a Data Boarding Pass encodes everything needed to deploy a specific Data Unlock use case:

| Boarding Pass Field | Data Boarding Pass Equivalent |
|---|---|
| **Airline** | Vertical sector (e.g., MSME, Agriculture, Healthcare) |
| **Flight Number** | Specific use case (e.g., Export Policy Intelligence) |
| **Passenger** | Data sources — the datasets flowing through this deployment |
| **Destination** | Purpose — the outcome this deployment is designed to achieve |
| **Gate / Seat** | Interface — how users access the intelligence (dashboard, copilot, API) |
| **Boarding Group** | Beneficiaries — the specific user personas served |

## Technical Manifestation: Specification

The Data Boarding Pass is a **specification**, not software. It is defined as a JSON Schema that any tool or platform can generate, validate, and consume. This is the correct design choice because:

1. **Multiple tools should be able to produce it.** A government agency's project management tool, a vendor's deployment automation, or a manual YAML editor should all be able to create a valid Data Boarding Pass.
2. **It is declarative, not executable.** It describes *what* a deployment looks like, not *how* to build it.
3. **It enables ecosystem-wide visibility.** A registry of Data Boarding Passes gives the Nodal Agency a complete view of all active Data Unlock deployments across a domain.

### Schema Definition

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://dataunlock.org/schemas/data-boarding-pass/v1.0.0",
  "title": "Data Boarding Pass",
  "description": "Deployment manifest for a Data Unlock use case",
  "type": "object",
  "required": ["version", "id", "sector", "useCase", "purpose", "dataSources", "beneficiaries", "interfaces", "nodalAgency"],
  "properties": {
    "version": {
      "type": "string",
      "const": "1.0.0",
      "description": "Schema version"
    },
    "id": {
      "type": "string",
      "format": "uri",
      "description": "Globally unique identifier for this Data Boarding Pass"
    },
    "sector": {
      "type": "object",
      "description": "The vertical domain this deployment serves",
      "required": ["name", "code"],
      "properties": {
        "name": { "type": "string" },
        "code": {
          "type": "string",
          "description": "Sector code from a recognized classification (e.g., ISIC Rev.4)"
        }
      }
    },
    "useCase": {
      "type": "object",
      "required": ["name", "description", "problemStatement"],
      "properties": {
        "name": { "type": "string" },
        "description": { "type": "string" },
        "problemStatement": {
          "type": "string",
          "description": "The specific problem this deployment addresses, as curated by the Problem Curator"
        }
      }
    },
    "purpose": {
      "type": "string",
      "description": "A concise statement of the intended outcome (e.g., 'Enable MSMEs to discover and evaluate export market opportunities')"
    },
    "sponsor": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "type": { "enum": ["government", "multilateral", "private", "ngo"] },
        "contact": { "type": "string", "format": "email" }
      }
    },
    "nodalAgency": {
      "type": "object",
      "required": ["name", "jurisdiction"],
      "properties": {
        "name": { "type": "string" },
        "jurisdiction": { "type": "string" },
        "registryEndpoint": { "type": "string", "format": "uri" }
      }
    },
    "dataSources": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["name", "provider", "dataPassportRef"],
        "properties": {
          "name": { "type": "string" },
          "provider": {
            "type": "object",
            "required": ["name", "role"],
            "properties": {
              "name": { "type": "string" },
              "role": { "const": "DIP" },
              "did": {
                "type": "string",
                "description": "Decentralised Identifier of the provider"
              }
            }
          },
          "dataPassportRef": {
            "type": "string",
            "format": "uri",
            "description": "URI to the Data Passport of this data source"
          },
          "accessMethod": {
            "enum": ["api", "bulk", "mcp", "a2a", "ftp"],
            "description": "How this data source is accessed"
          }
        }
      }
    },
    "dataUnlocker": {
      "type": "object",
      "required": ["name"],
      "properties": {
        "name": { "type": "string" },
        "platform": { "type": "string", "description": "Technology platform used (e.g., 'Google Data Commons', 'Custom')" },
        "deploymentModel": { "enum": ["centralised", "federated", "hybrid", "byod"] }
      }
    },
    "beneficiaries": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["persona", "description"],
        "properties": {
          "persona": { "type": "string" },
          "description": { "type": "string" }
        }
      }
    },
    "interfaces": {
      "type": "array",
      "minItems": 1,
      "items": {
        "type": "object",
        "required": ["type", "description"],
        "properties": {
          "type": {
            "enum": ["dashboard", "copilot", "api", "mcp_server", "agent", "report", "workflow"]
          },
          "description": { "type": "string" },
          "endpoint": { "type": "string", "format": "uri" }
        }
      }
    },
    "flightDescriptors": {
      "type": "object",
      "description": "Properties that this deployment guarantees about the data flowing through it",
      "properties": {
        "trusted": { "type": "boolean", "default": true },
        "harmonised": { "type": "boolean", "default": false },
        "linkable": { "type": "boolean", "default": false },
        "reusable": { "type": "boolean", "default": false },
        "machineReadable": { "type": "boolean", "default": false }
      }
    },
    "valueCainCoverage": {
      "type": "object",
      "description": "Which stages of the Data Unlock value chain this deployment implements",
      "properties": {
        "prepare": { "type": "boolean" },
        "connect": { "type": "boolean" },
        "enable": { "type": "boolean" },
        "apply": { "type": "boolean" }
      }
    },
    "metadata": {
      "type": "object",
      "properties": {
        "created": { "type": "string", "format": "date-time" },
        "lastUpdated": { "type": "string", "format": "date-time" },
        "status": { "enum": ["draft", "active", "deprecated", "archived"] }
      }
    }
  }
}
```

### Example: MSME Export Intelligence

```json
{
  "version": "1.0.0",
  "id": "urn:dataunlock:dbp:msme-export-v1",
  "sector": {
    "name": "MSME",
    "code": "ISIC-C"
  },
  "useCase": {
    "name": "Export Market Intelligence",
    "description": "Unified intelligence layer for MSMEs to discover and evaluate export opportunities",
    "problemStatement": "MSMEs struggle to discover export opportunities as trade, demand, and regulation data remain fragmented and hard to interpret."
  },
  "purpose": "Understand export markets, regulations, and tariff implications",
  "sponsor": {
    "name": "Ministry of Commerce (Central/State)",
    "type": "government"
  },
  "nodalAgency": {
    "name": "Ministry of Commerce",
    "jurisdiction": "India - Central & State"
  },
  "dataSources": [
    {
      "name": "Trade Statistics",
      "provider": { "name": "Ministry of Commerce", "role": "DIP" },
      "dataPassportRef": "urn:dataunlock:dp:moci-trade-stats-v2",
      "accessMethod": "api"
    },
    {
      "name": "Policy and Regulation",
      "provider": { "name": "Ministry of Commerce", "role": "DIP" },
      "dataPassportRef": "urn:dataunlock:dp:moci-policy-reg-v1",
      "accessMethod": "bulk"
    },
    {
      "name": "Tariff Data",
      "provider": { "name": "Australian Border Force", "role": "DIP" },
      "dataPassportRef": "urn:dataunlock:dp:abf-tariff-v3",
      "accessMethod": "api"
    }
  ],
  "dataUnlocker": {
    "name": "Trade Connect Platform",
    "platform": "Custom (Thurro + Trezix)",
    "deploymentModel": "hybrid"
  },
  "beneficiaries": [
    { "persona": "Entrepreneurs", "description": "MSME owners exploring export markets" },
    { "persona": "Trade Advisors", "description": "Government trade facilitation officers" }
  ],
  "interfaces": [
    { "type": "dashboard", "description": "Guided Dashboard with market analytics" },
    { "type": "copilot", "description": "AI Copilot for natural language queries about export regulations" }
  ],
  "flightDescriptors": {
    "trusted": true,
    "harmonised": true,
    "linkable": true,
    "reusable": true,
    "machineReadable": true
  },
  "valueCainCoverage": {
    "prepare": true,
    "connect": true,
    "enable": true,
    "apply": true
  },
  "metadata": {
    "created": "2026-01-19T00:00:00Z",
    "status": "active"
  }
}
```

## Registry

All Data Boarding Passes within an ecosystem should be registered with the Nodal Agency. The Nodal Agency maintains a **Boarding Pass Registry** — a searchable index that provides ecosystem-wide visibility into:

- Which use cases are active
- Which data sources are being consumed
- Which institutions are participating
- What interfaces are available to end users

The registry API should be a simple CRUD interface over the Data Boarding Pass schema, with search capabilities by sector, status, and participating DIP/DIU.
