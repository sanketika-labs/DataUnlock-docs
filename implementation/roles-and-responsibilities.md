# Roles & Responsibilities

This page provides a detailed RACI matrix (Responsible, Accountable, Consulted, Informed) for each activity in a Data Unlock deployment. Use this to map your organisation's teams to the required roles.

## RACI Matrix

### Phase 1: Assessment & Planning

| Activity | Sponsor | Problem Curator | DIP | Data Unlocker | DIU | Nodal Agency |
|---|---|---|---|---|---|---|
| Define use case & success metrics | **A** | **R** | C | C | C | I |
| Conduct data inventory | I | C | **R** | C | I | I |
| Perform gap analysis | I | C | C | **R** | I | **A** |
| Draft Data Boarding Pass | I | **R** | C | **R** | C | **A** |
| Assign institutional roles | **A** | C | C | C | C | **R** |

### Phase 2: Prepare Layer

| Activity | Sponsor | Problem Curator | DIP | Data Unlocker | DIU | Nodal Agency |
|---|---|---|---|---|---|---|
| Standardise data formats | I | I | **R** | C | I | I |
| Create schema descriptors | I | I | **R** | C | I | I |
| Deploy metadata catalogue | I | I | **R** | **R** | I | **A** |
| Encode ODRL access policies | I | C | **A** | **R** | I | C |
| Implement provenance (VCs) | I | I | **R** | C | I | **A** |
| Create Data Passports | I | I | **R** | **R** | I | **A** |
| Define Data Quality Scorecard | I | C | **R** | C | C | **A** |

### Phase 3: Connect Layer

| Activity | Sponsor | Problem Curator | DIP | Data Unlocker | DIU | Nodal Agency |
|---|---|---|---|---|---|---|
| Establish entity resolution | I | C | **R** | **R** | I | **A** |
| Create JSON-LD contexts | I | C | C | **R** | I | **A** |
| Deploy federated query layer | I | I | C | **R** | I | **A** |
| Maintain Mapping Registry | I | C | C | C | I | **R** |
| Define consent propagation rules | **A** | C | C | C | I | **R** |

### Phase 4: Enable + Apply

| Activity | Sponsor | Problem Curator | DIP | Data Unlocker | DIU | Nodal Agency |
|---|---|---|---|---|---|---|
| Build embedding pipeline | I | I | C | **R** | I | I |
| Construct knowledge graph | I | C | C | **R** | I | I |
| Deploy MCP server | I | I | C | **R** | I | I |
| Build user interfaces | C | **R** | I | **R** | C | I |
| Implement citation/grounding | I | I | I | **R** | I | **A** |
| Deploy audit logging | I | I | I | **R** | I | **A** |
| User acceptance testing | I | C | I | C | **R** | I |

### Ongoing Operations

| Activity | Sponsor | Problem Curator | DIP | Data Unlocker | DIU | Nodal Agency |
|---|---|---|---|---|---|---|
| Monitor data quality | I | I | **R** | C | I | **A** |
| Manage embedding versions | I | I | I | **R** | I | I |
| Onboard new DIPs | C | C | **R** | C | I | **A** |
| Enforce compliance | I | I | I | I | I | **R** |
| Measure outcomes | **A** | **R** | I | C | **R** | I |
| Evolve specifications | C | C | C | C | C | **R** |

---

## Team Composition Guide

### For a Data Information Provider (DIP)

| Role | Responsibility | Estimated effort |
|---|---|---|
| Data Steward | Certifies data quality, defines access policies, maintains Data Passports | 50% FTE ongoing |
| Data Engineer | Standardises formats, creates schemas, deploys catalogue endpoint | 1-2 FTEs during setup |
| Security/Compliance Officer | Reviews ODRL policies, manages DID keys, signs VCs | 25% FTE ongoing |

### For a Data Unlocker

| Role | Responsibility | Estimated effort |
|---|---|---|
| Solution Architect | Designs the overall deployment, selects technology stack | 1 FTE during setup |
| Data Engineers | Builds ingestion pipelines, embedding pipelines, knowledge graph | 2-4 FTEs during setup |
| ML/AI Engineer | Configures embedding models, builds RAG pipeline, deploys MCP server | 1-2 FTEs during setup |
| Frontend Developer | Builds dashboards, copilot interfaces | 1-2 FTEs during Apply phase |
| DevOps/Platform Engineer | Deploys and operates infrastructure | 1 FTE ongoing |

### For a Nodal Agency

| Role | Responsibility | Estimated effort |
|---|---|---|
| Standards Lead | Defines and evolves interoperability specifications | 1 FTE ongoing |
| Compliance Officer | Audits conformance, enforces governance | 1 FTE ongoing |
| Registry Administrator | Operates the catalogue, mapping registry, and Boarding Pass registry | 0.5 FTE ongoing |
| Ecosystem Manager | Onboards new participants, resolves disputes | 1 FTE ongoing |
