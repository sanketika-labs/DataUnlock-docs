# Reference: Invest UP

This reference illustration shows how Data Unlock is applied to investor facilitation — helping state investment agencies build holistic investor profiles and accelerate decision-making.

---

## Context

### The Problem

Investor data sits across multiple disconnected systems — CRM platforms, enterprise databases, state investment portals, and third-party data providers. State investment facilitation agencies (and their field officers, known as "Udyami Mitras") cannot assemble a complete picture of an investor quickly enough to respond to their needs.

### The Goal

Build a unified intelligence layer that enables investment facilitation officers to instantly access holistic investor profiles, risk assessments, and recommended actions — reducing response times from weeks to minutes.

---

## Business Architecture Mapping

| Data Unlock Role | Invest UP Implementation |
|---|---|
| **Sponsor** | Invest UP (Uttar Pradesh investment agency) |
| **Problem Curator** | Invest UP leadership |
| **DIPs** | Invest UP (investor records), Thurro (enterprise intelligence), Investors (self-reported data) |
| **Data Unlocker** | Implementation partner |
| **DIUs** | Udyami Mitras (account managers / field officers) |
| **Nodal Agency** | Invest UP |

---

## Data Boarding Pass

```json
{
  "version": "1.0.0",
  "id": "urn:dataunlock:dbp:invest-up-v1",
  "sector": { "name": "Investment Facilitation", "code": "ISIC-K" },
  "useCase": {
    "name": "Investor Intelligence & Case Management",
    "description": "Unified investor profiles with risk assessment and AI-assisted case management",
    "problemStatement": "Investor data sits across multiple systems, preventing agencies from assembling, understanding, and responding to investor needs quickly"
  },
  "purpose": "Build and maintain holistic investor profiles and enable faster decisions",
  "nodalAgency": { "name": "Invest UP", "jurisdiction": "Uttar Pradesh, India" },
  "dataSources": [
    {
      "name": "Investor Records",
      "provider": { "name": "Invest UP", "role": "DIP" },
      "accessMethod": "api"
    },
    {
      "name": "Enterprise Intelligence",
      "provider": { "name": "Thurro", "role": "DIP" },
      "accessMethod": "api"
    },
    {
      "name": "Investor Self-Reported Data",
      "provider": { "name": "Investors (via portal)", "role": "DIP" },
      "accessMethod": "api"
    }
  ],
  "beneficiaries": [
    { "persona": "Udyami Mitras", "description": "Field officers and account managers responsible for investor facilitation" },
    { "persona": "Agency Leadership", "description": "Decision-makers monitoring investment pipeline and outcomes" }
  ],
  "interfaces": [
    { "type": "agent", "description": "AI-Assisted Case Management Portal with automated investor profiling" },
    { "type": "dashboard", "description": "Investment pipeline analytics dashboard" }
  ]
}
```

---

## Value Chain Implementation

### Prepare

| Activity | Implementation |
|---|---|
| Investor records | Extracted from Invest UP's existing systems, standardised into Parquet with consistent naming |
| Enterprise data | Ingested from Thurro's enterprise intelligence API, normalised to shared schema |
| Self-reported data | Collected via investor portal, validated against enterprise data |
| Data quality | Cross-validation of investor details against multiple sources to score completeness and accuracy |

### Connect

| Activity | Implementation |
|---|---|
| Entity resolution | Investors matched across systems using GSTIN, PAN, or hashed identifiers |
| Semantic alignment | Investor profile fields mapped to Schema.org (`Organization`, `Person`, `PostalAddress`) |
| Profile assembly | Unified investor profile assembled from multiple sources, with conflict resolution rules |

### Enable

| Activity | Implementation |
|---|---|
| Risk profiling | ML models trained on historical investor data to generate risk scores and investment likelihood |
| Knowledge Graph | Investor → Enterprise → Sector → Geography graph for relationship analysis |
| MCP Server | Exposes tools: `get_investor_profile`, `assess_risk`, `recommend_actions`, `search_investors` |

### Apply

**Interface: AI-Assisted Case Management Portal**

A case management interface where Udyami Mitras can:

- View a holistic investor profile assembled from all data sources
- See AI-generated risk assessments with supporting evidence
- Receive recommended next actions (follow-up calls, document requests, escalations)
- Track their case pipeline with automated status updates

**Interface: Pipeline Dashboard**

An executive dashboard showing investment pipeline metrics, sector-wise analysis, and performance tracking for Udyami Mitras.

---

## Key Takeaways for Implementers

1. **Entity resolution with Indian business identifiers** — GSTIN and PAN serve as natural join keys for enterprise data across government and private systems. Where these are unavailable, probabilistic matching on name + address + sector provides a fallback.

2. **Privacy in multi-source assembly** — When assembling profiles from multiple sources (including investor self-reported data and third-party intelligence), the Data Passport's ODRL policies must encode strict purpose-binding: the unified profile is for investment facilitation only, not for surveillance or credit scoring.

3. **AI-assisted, not AI-replaced** — The Apply layer surfaces recommendations and risk scores, but the Udyami Mitra makes the final decision. The citation/grounding schema ensures that every AI recommendation can be traced to specific data sources, maintaining human accountability.

4. **State-level deployment** — This example demonstrates that Data Unlock can be deployed at the state level (not just national), with the state agency serving as both Sponsor and Nodal Agency.
