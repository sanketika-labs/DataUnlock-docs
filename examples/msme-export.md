# Reference: MSME Export Intelligence

This reference implementation shows how Data Unlock is applied to help Micro, Small, and Medium Enterprises (MSMEs) discover and evaluate export market opportunities by linking trade statistics, regulatory data, and tariff information.

> **Note:** This is an illustrative example based on an active implementation track.

---

## Context

### The Problem

MSMEs struggle to discover export opportunities because the data they need is fragmented across multiple agencies and formats:

- **Trade statistics** sit with the Ministry of Commerce
- **Export policies and regulations** are spread across multiple departmental websites and PDF documents
- **Tariff data** comes from destination country border agencies (e.g., Australian Border Force)
- **Government schemes** for exporters are managed by the Ministry of MSME

An entrepreneur who wants to understand whether exporting a specific product to a specific market is viable must manually navigate all of these sources, often without the technical expertise to interpret the data.

---

## Business Architecture Mapping

| Data Unlock Role | MSME Export Implementation |
|---|---|
| **Sponsor** | Ministry of Commerce (Central/State) |
| **Problem Curator** | Trade bodies (Central/State) |
| **DIPs** | Trade Statistics (Commerce), Policy & Regulation (Commerce), Tariff data (destination country border agencies), Schemes (MSME), Trade Connect platform (Commerce) |
| **Data Unlocker** | Trade Connect (Ministry of Commerce), Thurro, Trezix |
| **DIUs** | Entrepreneurs (MSME owners) |
| **Nodal Agency** | Ministry of Commerce (Central/State) |

---

## Data Boarding Pass

```json
{
  "version": "1.0.0",
  "id": "urn:dataunlock:dbp:msme-export-v1",
  "sector": { "name": "MSME", "code": "ISIC-C" },
  "useCase": {
    "name": "Export Market Intelligence",
    "description": "Unified intelligence for MSMEs to discover, evaluate, and pursue export opportunities",
    "problemStatement": "MSMEs struggle to discover export opportunities as trade, demand, and regulation data remain fragmented"
  },
  "purpose": "Understand export markets, regulations, and tariff implications",
  "nodalAgency": { "name": "Ministry of Commerce", "jurisdiction": "India - Central & State" },
  "dataSources": [
    {
      "name": "Trade Statistics",
      "provider": { "name": "Ministry of Commerce", "role": "DIP" },
      "accessMethod": "api"
    },
    {
      "name": "Policy and Regulation Documents",
      "provider": { "name": "Ministry of Commerce", "role": "DIP" },
      "accessMethod": "bulk"
    },
    {
      "name": "Tariff Data",
      "provider": { "name": "Destination Country Border Agencies", "role": "DIP" },
      "accessMethod": "api"
    },
    {
      "name": "MSME Schemes",
      "provider": { "name": "Ministry of MSME", "role": "DIP" },
      "accessMethod": "bulk"
    }
  ],
  "beneficiaries": [
    { "persona": "Entrepreneurs", "description": "MSME owners exploring and evaluating export markets" },
    { "persona": "Trade Advisors", "description": "Government officials facilitating MSME exports" }
  ],
  "interfaces": [
    { "type": "dashboard", "description": "Guided Dashboard with market analytics and tariff comparisons" },
    { "type": "copilot", "description": "AI Copilot for natural language queries about export regulations and market opportunities" }
  ]
}
```

---

## Value Chain Implementation

### Prepare

| Activity | Implementation |
|---|---|
| Trade statistics | Standardised from Commerce Ministry APIs into Parquet with HS code schema |
| Policy documents | Extracted from PDFs, structured into JSON with metadata (policy number, applicable HS codes, effective date, jurisdiction) |
| Tariff data | Ingested via API from destination country systems, normalised to common schema |
| MSME schemes | Catalogued with eligibility criteria, application process, and linked sector codes |

All data products are catalogued in a DCAT-AP endpoint, with ODRL policies (purpose: `exportIntelligence`, `msmeSupport`) and W3C VCs for provenance.

### Connect

| Activity | Implementation |
|---|---|
| Entity resolution | HS codes serve as the primary join key across trade statistics, tariff data, and policy documents |
| Semantic alignment | JSON-LD contexts map local fields to Schema.org + WCO nomenclature |
| Knowledge Graph | A product-market-regulation graph links HS codes → countries → tariff rates → applicable policies → available schemes |

The Knowledge Graph enables queries like: "For HS code 0901 (Coffee), what are the tariff rates in Australia, and are there any Indian export schemes that apply?"

### Enable

| Activity | Implementation |
|---|---|
| Embeddings | Policy and regulation documents are chunked, embedded, and stored in a vector database |
| Knowledge Graph | The product-market-regulation graph is queryable via GraphQL |
| Metadata Bridge | Links KG entities (Policy #52 → HS 8532) to vector store chunks containing the policy full text |
| MCP Server | Exposes tools: `query_trade_statistics`, `get_tariff_rates`, `find_applicable_policies`, `search_schemes` |

### Apply

**Interface 1: Guided Dashboard**
A visual analytics interface where entrepreneurs can explore export markets by product, see tariff comparisons across destination countries, and identify trending markets.

**Interface 2: AI Copilot**
A conversational interface where entrepreneurs can ask questions in natural language:

- "What are the requirements for exporting organic spices to Germany?"
- "Compare tariff rates for HS 0901 across ASEAN countries"
- "Are there any government schemes for first-time exporters in Karnataka?"

All responses are grounded with citations pointing to the specific trade statistics, policy documents, or tariff databases that support the answer.

---

## Key Takeaways for Implementers

1. **HS codes as the universal join key** — In trade intelligence use cases, the Harmonised System code serves as a natural entity resolution mechanism across trade statistics, tariff databases, and policy documents.

2. **PDF-to-vector pipeline is critical** — Much of the regulatory intelligence lives in PDF documents. The Prepare + Enable pipeline (PDF → structured JSON → embeddings → vector store) is the most impactful technical investment.

3. **Cross-jurisdiction data federation** — Tariff data comes from destination country systems. The Data Unlock architecture handles this naturally — each country's tariff API is a DIP, and the Data Unlocker federates across them.

4. **Dual interface strategy** — The combination of a guided dashboard (for exploration) and an AI copilot (for specific questions) serves different user needs within the same persona.
