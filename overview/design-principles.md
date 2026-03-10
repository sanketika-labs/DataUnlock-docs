# Design Principles

Data Unlock is built on five non-negotiable design principles. Any implementation that violates these principles is not a conforming Data Unlock deployment.

## 1. Federated by Default

Data remains with its custodian institution. There is no requirement to copy, migrate, or centralise data into a shared repository. The architecture assumes that each Data Information Provider (DIP) operates its own infrastructure and exposes data through standardised interfaces.

**What this means in practice:**
- No central data lake is required (though one may exist as an optimisation)
- Each DIP maintains sovereignty over storage, compute, and access control
- Interoperability is achieved through shared protocols, not shared databases
- A query may be answered by federating across multiple DIPs in real-time

## 2. Trust before Intelligence

Governance, quality, and consent are non-negotiable prerequisites. Before any dataset enters the Data Unlock ecosystem, it must pass through the Prepare layer: cleaned, standardised, certified for provenance, governed by explicit access policies, and catalogued with rich metadata.

**What this means in practice:**
- No dataset is "enriched" or made "AI-ready" until its provenance is verifiable
- Access policies (encoded in ODRL) travel with the data, not just with the system
- Every data product has a named owner who has explicitly certified its quality
- Consent tokens are validated at the point of access, not assumed

## 3. Semantic-first Integration

Meaning and context must be established before analytics or AI can operate. Two datasets cannot be "joined" unless their fields are mapped to shared ontologies. An embedding is useless if the consuming system doesn't know what vocabulary produced it.

**What this means in practice:**
- Every field in a data product must link to a URI from a recognised vocabulary (Schema.org, domain-specific ontologies)
- Entity resolution uses Decentralised Identifiers (DIDs) or hashed common keys
- Semantic alignment is enforced as a prerequisite for the Connect layer, not treated as an optional enhancement
- Cross-domain vocabulary collisions are resolved through a shared Mapping Registry

## 4. AI-readiness as a Progression

The journey from human-readable data to machine-interpretable intelligence is a progression, not a binary state. Data Unlock defines four stages (Prepare → Connect → Enable → Apply), and each stage adds a layer of machine readiness.

**What this means in practice:**
- An institution can begin with just the Prepare layer and derive immediate value (discoverability, governance)
- AI capabilities (embeddings, knowledge graphs, MCP interfaces) are added incrementally as the Connect and Enable layers are implemented
- There is no requirement to implement all four layers simultaneously
- Each layer's specifications are independently implementable

## 5. Reuse over Rebuild

The same backbone of governance, semantics, and AI-readiness services supports multiple use cases. When a new vertical (e.g., healthcare, education) is onboarded, the core architecture remains the same — only the purpose, data sources, and interfaces change.

**What this means in practice:**
- Core building blocks (data catalogue, consent service, entity resolution, embedding pipeline, MCP server) are deployed once and shared across use cases
- A new Data Boarding Pass (use-case deployment) reuses existing infrastructure rather than building a parallel stack
- Standards and protocols are domain-agnostic; domain-specific content lives in ontologies and vocabularies, not in the platform code
