# Apply: Outcomes & Impact

**Goal:** Convert AI-ready data into real-world decisions, insights, and automated actions through user-facing interfaces.

The Apply layer is where the value chain delivers measurable outcomes. All intelligence surfaced through this layer must be traceable back to specific trusted datasets through the citation and grounding mechanisms specified below.

---

## Interact: Conversational Interfaces

### Specification

Conversational AI interfaces (chatbots, copilots, RAG assistants) **must** implement citation and grounding schemas that trace every response back to its source data.

| Attribute | Requirement |
|---|---|
| **Citation standard** | Every factual claim in a response must include a `source_ref` pointing to the Data Passport URI and (where applicable) the specific chunk ID |
| **Grounding schema** | Responses must include a structured `grounding` object that lists all data sources consulted |
| **Explainability** | Users must be able to "drill down" from any cited claim to the underlying data product |
| **Multi-language** | Conversational interfaces should support queries in multiple languages relevant to the deployment jurisdiction |

### Why mandatory citations?

This is not optional polish — it is the mechanism that makes Data Unlock trustworthy for policy decisions. When a policymaker asks "What is the trend in cotton exports to Southeast Asia?" and the copilot answers, the policymaker must be able to verify that the answer comes from MoCI trade statistics (certified, provenance-verified) rather than from the LLM's training data (which may be outdated or hallucinated).

### Citation Schema

```json
{
  "@type": "GroundedResponse",
  "query": "What are the top 5 export products from Karnataka to Australia?",
  "response": "Based on 2024-2025 trade statistics, the top 5 export products from Karnataka to Australia are...",
  "grounding": {
    "sources": [
      {
        "dataPassportRef": "urn:dataunlock:dp:moci-trade-stats-v2",
        "dataProductName": "India Trade Statistics",
        "publisher": "Ministry of Commerce",
        "freshness": "2025-12-31",
        "provenanceCredential": "urn:dataunlock:vc:moci-trade-stats-v2-2026-01"
      }
    ],
    "chunks": [
      {
        "chunkId": "urn:dataunlock:chunk:moci-trade-stats-v2:karnataka-au-2024",
        "relevanceScore": 0.94
      }
    ]
  },
  "citations": [
    {
      "claim": "Engineering goods constituted 34% of Karnataka's exports to Australia",
      "source_ref": "urn:dataunlock:dp:moci-trade-stats-v2",
      "evidence": "HS chapters 84-85, Karnataka origin, Australia destination, FY 2024-25"
    }
  ],
  "confidence": 0.91,
  "disclaimer": "This response is grounded in official trade statistics. Verify critical decisions with the source data."
}
```

---

## Analyse: Cross-Domain Insights

### Specification

Analytical interfaces (dashboards, reports, statistical tools) **must** expose data through open analytical APIs that support cross-source analysis.

| Attribute | Requirement |
|---|---|
| **API standard** | [OData v4.01](https://www.odata.org/documentation/) for tabular analytical queries; GraphQL for relationship-based analysis |
| **Aggregation** | APIs must support server-side aggregation (GROUP BY, SUM, AVG, COUNT) to minimise data transfer |
| **Filtering** | Standard filter expressions (`$filter` in OData, `where` clauses in GraphQL) |
| **Cross-source** | Dashboard queries may span multiple data products; the analytical API must handle federation transparently |
| **Export** | Results must be exportable in CSV, JSON, and Parquet formats |

### Dashboard Architecture Pattern

```
┌─────────────────────────────────────────┐
│          Dashboard / Report UI           │
└──────────────────┬──────────────────────┘
                   │ OData / GraphQL
                   ▼
┌─────────────────────────────────────────┐
│         Analytical API Gateway           │
│  (query routing, aggregation, caching)   │
└──────┬──────────┬──────────────┬────────┘
       │          │              │
       ▼          ▼              ▼
  ┌─────────┐ ┌─────────┐ ┌──────────┐
  │ Data    │ │Knowledge│ │ Vector   │
  │Warehouse│ │ Graph   │ │ Store    │
  └─────────┘ └─────────┘ └──────────┘
```

---

## Decide/Act: Workflow Automation

### Specification

Data Unlock deployments that require automated responses to data events **must** implement event-driven workflows using CloudEvents.

| Attribute | Requirement |
|---|---|
| **Event format** | [CloudEvents v1.0](https://cloudevents.io/) |
| **Transport** | HTTPS webhooks (primary), Apache Kafka (for high-throughput), or AMQP |
| **Event types** | Standard event types for data lifecycle events: `dataunlock.data.published`, `dataunlock.data.updated`, `dataunlock.alert.threshold`, `dataunlock.alert.anomaly` |
| **Workflow engine** | Any engine that can subscribe to CloudEvents and execute actions (e.g., Apache Airflow, Temporal, AWS Step Functions, custom) |

### Why CloudEvents?

CloudEvents provides a standard envelope for event data, ensuring that a workflow trigger from DIP A can be consumed by Data Unlocker B's automation engine without custom adapters. This enables scenarios like:

- DIP A publishes updated trade statistics → Data Unlocker B's embedding pipeline automatically re-indexes
- An anomaly detection model identifies a sudden spike in export rejections → An alert event triggers a review workflow at the relevant trade body
- A risk model identifies high-risk investor profiles → A case management system creates review tickets for account managers

### Example: CloudEvent

```json
{
  "specversion": "1.0",
  "type": "dataunlock.alert.threshold",
  "source": "urn:dataunlock:dbp:msme-export-v1",
  "id": "evt-2026-03-10-001",
  "time": "2026-03-10T08:30:00Z",
  "subject": "urn:dataunlock:dp:moci-trade-stats-v2",
  "datacontenttype": "application/json",
  "data": {
    "alert": "Export value for HS code 0901 to EU dropped 40% month-over-month",
    "metric": "export_value_usd",
    "threshold": -0.30,
    "actual": -0.40,
    "recommendation": "Review EU tariff changes for HS 0901 (Coffee)",
    "groundingRef": "urn:dataunlock:dp:moci-trade-stats-v2"
  }
}
```

---

## Recommendation Engines

Data Unlock deployments **may** include recommendation engines that convert patterns and insights into proactive, actionable suggestions.

| Capability | Description |
|---|---|
| **Scheme matching** | Recommend government schemes to enterprises based on their profile and eligibility criteria |
| **Market opportunity** | Suggest export markets based on product-market fit analysis across trade data |
| **Risk alerts** | Proactively notify users of regulatory changes, tariff adjustments, or compliance deadlines |
| **Compliance tracking** | Monitor ongoing compliance status and flag upcoming requirements |

Recommendation engines consume the same AI-ready data (embeddings, knowledge graph, vector store) exposed through the Enable layer. Their outputs should be surfaced through the same grounded citation mechanism specified for conversational interfaces.

---

## Output Traceability Requirements

All Apply layer interfaces — whether conversational, analytical, or automated — **must** maintain a complete audit trail:

| Requirement | Description |
|---|---|
| **Query logging** | Every query (NLQ, API call, dashboard interaction) must be logged with timestamp, requester identity, and purpose |
| **Response provenance** | Every response must be traceable to the specific data products, chunks, and knowledge graph nodes that produced it |
| **Consent validation** | The audit log must record that consent/policy was validated for each data access in the response chain |
| **Retention** | Audit logs must be retained for the period specified by the Nodal Agency (minimum 1 year) |
