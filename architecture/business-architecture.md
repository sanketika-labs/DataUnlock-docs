# Business Architecture

The Data Unlock business architecture defines the roles, relationships, and responsibilities of the entities that participate in the ecosystem. It is deliberately modelled on established DPI patterns — separating the roles of data provision, data transformation, data consumption, and ecosystem governance.

## Roles

### Sponsor

The entity that initiates and funds a Data Unlock use case to drive measurable policy, economic, or social outcomes.

- Typically a government ministry, development agency, or multilateral body
- Defines the high-level purpose and success metrics
- Does not necessarily participate in the technical implementation

**Example:** Ministry of Commerce sponsoring an MSME Export Intelligence use case.

### Single Curator of Problems

For each domain, a designated institution or consortium curates and prioritises the problem statements that Data Unlock will address.

- Ensures that problems reflect real-world needs, not technology-driven solutions
- Reduces duplication of effort across agencies
- Maintains alignment with agreed domain or national priorities
- Typically a sector-specific body, industry consortium, or standards organisation

**Example:** Trade bodies (central and state level) curating problem statements for export intelligence.

### Data Information Provider (DIP)

The institutional custodian of public and/or private data. DIPs are the **source** of raw data that enters the Data Unlock ecosystem.

- Stewards data and ensures quality, privacy, and controlled access
- Exposes data through standardised interfaces defined in the [Prepare specification](../specifications/prepare-layer.md)
- Retains full sovereignty over their data — nothing is forcibly centralised
- Responsible for certifying provenance and maintaining lineage

**Example:** Ministry of Statistics (MoSPI), Ministry of Agriculture, Trade Statistics division.

### Data Unlocker (DU)

The implementation partner that integrates, standardises, and links multi-source data into interoperable, AI-ready intelligence. This is the technical execution role.

- Consumes data from one or more DIPs
- Implements the Prepare, Connect, Enable, and Apply layers
- Operates the technical infrastructure (knowledge graphs, vector stores, MCP servers, dashboards)
- May be a government IT wing, a contracted system integrator, or an open-source platform operator

**Example:** Google Data Commons, a state IT agency, or a private vendor like Thurro/Trezix.

### Data Information User (DIU)

The end consumer of unlocked intelligence. DIUs consume insights through dashboards, copilots, APIs, or applications to make informed decisions.

- Does not interact with raw data — only with processed, governed, AI-ready outputs
- May be policymakers, researchers, entrepreneurs, citizens, or AI agents
- Access is governed by purpose-binding policies set in the Authorise step

**Example:** Entrepreneurs seeking export market intelligence, policymakers monitoring development indicators.

### Nodal Agency / Orchestrator

The entity that acts as the ecosystem coordinator. The Nodal Agency does **not** centralise raw data — instead, it manages the rules of engagement.

- Onboards DIPs and DIUs into the ecosystem
- Enforces governance and consent frameworks
- Manages interoperability standards and compliance
- Ensures auditability and data safety
- Operates (or delegates operation of) shared services like the metadata catalogue, entity resolution registry, and consent service

**Example:** MoSPI for statistical data, Ministry of Commerce for trade data.

## Interaction Model

```
                    ┌─────────────┐
                    │   Sponsor   │
                    └──────┬──────┘
                           │ funds & defines purpose
                           ▼
┌──────────┐    ┌─────────────────────┐    ┌──────────────────────┐
│   DIP    │◄──►│    Data Unlocker    │◄──►│   Problem Curator    │
│ (Source) │    │  (Implementation)   │    │  (Domain Authority)  │
└──────────┘    └─────────┬───────────┘    └──────────────────────┘
                          │
                          │ delivers intelligence
                          ▼
                    ┌───────────┐
                    │    DIU    │
                    │ (Consumer)│
                    └───────────┘

        ┌───────────────────────────────────┐
        │       Nodal Agency / Orchestrator │
        │  (Governance, Standards, Audit)   │
        └───────────────────────────────────┘
```

The Nodal Agency oversees all interactions but does not sit in the data path. Data flows directly from DIPs to Data Unlockers to DIUs. The Nodal Agency enforces the rules under which those flows operate.

## One Architecture, Many Use Cases

A critical design property of Data Unlock is that the **core architecture remains the same** across use cases. When deploying a new vertical (e.g., moving from trade intelligence to health surveillance), only three things change:

| What changes | What does not |
|---|---|
| Purpose (the problem being solved) | Core architecture |
| Data sources (which DIPs participate) | Building blocks (catalogue, consent, entity resolution, etc.) |
| Interfaces (dashboards, copilots, workflows) | Trust and access controls |

This "reuse over rebuild" principle is what makes Data Unlock viable as population-scale DPI.
