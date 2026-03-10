# Reference: MoSPI Data Commons

This is a reference implementation case study showing how the Data Unlock architecture is being applied to India's Ministry of Statistics and Programme Implementation (MoSPI) using Google Data Commons as the Data Unlocker platform.

> **Note:** This is an illustrative example. Specific technical details may evolve as the implementation progresses.

---

## Context

### The Challenge

Various agencies across India — MoSPI, Ministry of MSME, Ministry of Commerce & Industry, and others — house vast datasets (both structured and unstructured). These are often fragmented across disparate PDFs, tables, and siloed databases, making cross-ministry analysis extremely difficult.

### The Friction

Current access mechanisms require deep technical expertise. Policymakers and citizens struggle to simply "ask the data a question." Assembling a cross-ministry dataset for policy analysis can take months of manual effort.

### The Goal

Build an interoperable, AI-ready infrastructure that democratises access to India's official statistics — enabling natural language queries, cross-ministry linking, and AI-grounded policy insights.

---

## Business Architecture Mapping

| Data Unlock Role | MoSPI Implementation |
|---|---|
| **Sponsor** | MoSPI |
| **Problem Curator** | Sector-wise institution bodies (Agriculture, MSME, Textile) |
| **Data Information Providers (DIPs)** | MoSPI, Ministry of Agriculture, Ministry of Commerce, Ministry of MSME |
| **Data Unlocker** | Google Data Commons (platform) |
| **Data Information Users (DIUs)** | Policymakers |
| **Nodal Agency** | MoSPI |

---

## Technical Architecture

### Platform: Data Commons

The implementation uses Google Data Commons as the core Data Unlocker platform. Data Commons provides:

- **Knowledge Graph** — Aggregates data into a single semantic graph using Schema.org ontology. Entities like "District X" become defined objects with properties, replacing disconnected text strings across different databases.
- **Auto-Schematisation** — LLM-powered mapping of local CSVs and datasets to Schema.org standards automatically, reducing the traditional ETL cycle from months to days.
- **SDMX Interoperability** — Bridges legacy statistical standards (SDMX, widely used by MoSPI and international statistical agencies) with modern AI-ready graph structures.
- **MCP Server** — Exposes the knowledge graph as tools that AI agents can query directly.
- **NL Query & Visualisation** — Users can ask questions in plain English and receive instant charts, maps, and tables without coding.

### Data Sovereignty: BYOD Model

The deployment follows the BYOD (Bring Your Own Data) model:

- A custom Data Commons instance runs on MoSPI's own cloud project (Docker/Kubernetes)
- The Knowledge Graph sits in a private cloud instance
- Sensitive data never leaves the firewall
- The custom instance automatically federates with the public Data Commons graph to pull in context (global GDP, population, international comparisons) without hosting that data locally

### Component Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                     Agentic Apps                              │
│               ┌──────────────┐  ┌──────────────┐            │
│               │ Vyapar Sah   │  │  Yojan (AI)  │            │
│               │    (AI)      │  │              │            │
│               └──────┬───────┘  └──────┬───────┘            │
│                      │                 │                     │
│   ┌──────────────────▼─────────────────▼──────────────┐     │
│   │         Custom Data Commons Instance               │     │
│   │  ┌────────────────────────────────────────────┐   │     │
│   │  │  Conversational Agent                       │   │     │
│   │  │  Search, Insights, Data Exploration         │   │     │
│   │  └────────────────────┬───────────────────────┘   │     │
│   │                       │                            │     │
│   │  ┌────────────────────▼───────────────────────┐   │     │
│   │  │     India KG (CloudSQL / SpannerGraph)      │   │     │
│   │  └────────────────────┬───────────────────────┘   │     │
│   │                       │                            │     │
│   │  ┌────────────────────▼───────────────────────┐   │     │
│   │  │  Custom DC Mixer (APIs) + MCP Server        │   │     │
│   │  └────────────────────┬───────────────────────┘   │     │
│   └───────────────────────┼────────────────────────────┘     │
│                           │                                   │
└───────────────────────────┼───────────────────────────────────┘
                            │ Federation
                            ▼
                   ┌─────────────────┐
                   │  Public Data    │
                   │  Commons KG     │
                   │  (Global stats) │
                   └─────────────────┘
```

### Intelligence Layer

The intelligence layer sits between the Knowledge Graph and the user-facing applications:

- **Orchestrator Agent** — Custom code (Python/Node.js) that understands user intent and routes queries to the appropriate data source
- **Vector Store** — Stores the actual text content of policy PDFs, enabling RAG-grounded responses
- **Metadata Bridge** — The Knowledge Graph knows that "Policy #52 exists and applies to HS Code 8532." The Vector Store holds the full text. The agent uses the Graph ID to fetch the correct text.

---

## Value Chain Coverage

| Stage | Implementation Status |
|---|---|
| **Prepare** | Auto-schematisation maps raw MoSPI CSVs to Schema.org. SDMX interoperability bridges legacy formats. |
| **Connect** | Knowledge Graph provides entity resolution and semantic linking across ministries. Schema.org ontology ensures shared meaning. |
| **Enable** | MCP Server enables AI agent access. Knowledge Graph is queryable via SPARQL and NL query. |
| **Apply** | Conversational agent (search, insights, exploration), analytical dashboards, domain-specific agentic apps (Vyapar Sah, Yojan). |

---

## Data Boarding Pass

```json
{
  "version": "1.0.0",
  "id": "urn:dataunlock:dbp:mospi-data-commons-v1",
  "sector": { "name": "National Statistics", "code": "ISIC-O" },
  "useCase": {
    "name": "Policy Intelligence Platform",
    "description": "Cross-ministry statistical intelligence for evidence-based policymaking",
    "problemStatement": "Policymakers cannot link administrative and survey data across ministries in time to inform decisions"
  },
  "purpose": "Monitor progress, identify gaps, and take data-driven decisions",
  "sponsor": { "name": "MoSPI", "type": "government" },
  "nodalAgency": { "name": "MoSPI", "jurisdiction": "India" },
  "dataSources": [
    { "name": "MoSPI Statistical Datasets", "provider": { "name": "MoSPI", "role": "DIP" }, "accessMethod": "bulk" },
    { "name": "Agricultural Statistics", "provider": { "name": "Ministry of Agriculture", "role": "DIP" }, "accessMethod": "api" },
    { "name": "MSME Registry Data", "provider": { "name": "Ministry of MSME", "role": "DIP" }, "accessMethod": "api" }
  ],
  "dataUnlocker": { "name": "Google Data Commons", "platform": "Data Commons", "deploymentModel": "byod" },
  "beneficiaries": [
    { "persona": "Policy Makers", "description": "Central and state government officials making evidence-based decisions" },
    { "persona": "Researchers", "description": "Academic and institutional researchers analysing national indicators" }
  ],
  "interfaces": [
    { "type": "dashboard", "description": "Analytical & Insight Dashboard" },
    { "type": "copilot", "description": "Conversational search, insights, and data exploration agent" },
    { "type": "mcp_server", "description": "MCP endpoint for AI agent integration" }
  ]
}
```

---

## Key Takeaways for Implementers

1. **Schema.org as the semantic backbone** — The MoSPI implementation demonstrates that Schema.org (a widely adopted, general-purpose ontology) can serve as the primary vocabulary for government statistical data, reducing the need for custom ontologies.

2. **BYOD for data sovereignty** — The containerised deployment model ensures that sensitive government data never leaves the institution's infrastructure while still benefiting from federation with global public data.

3. **Auto-schematisation as an accelerator** — LLM-powered mapping of local schemas to Schema.org dramatically reduces the manual effort of the Connect layer (from months to days for schema alignment).

4. **Federation over centralisation** — The custom instance federates with the public Data Commons graph for international context without duplicating that data locally.
