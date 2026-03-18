# MoSPI MCP Server: Suggested Improvements

**Based on:** Thurro evaluation report (Feb 2026), Data Unlock specification framework, and the esankhyiki-mcp GitHub repository.

This document maps the gaps identified in MoSPI's current MCP implementation against the Data Unlock specification and proposes concrete improvements. The recommendations are structured by the Data Unlock layer they align with, making it clear *why* each change matters.

---

## 1. Prepare Layer Alignment

The Prepare layer defines the foundational standards every data product must meet before it enters the ecosystem: discoverability, schema descriptors, data curation, access policies, and provenance. MoSPI's MCP server currently addresses none of these formally.

### 1.1 Expose a DCAT-AP Catalogue with API Discovery

**Current state:** The MCP server's `step1_know_about_mospi_api()` returns a flat list of 19 dataset names. There is no machine-readable catalogue. External systems cannot discover MoSPI's datasets without already knowing the MCP endpoint.

**What the spec requires:** Every Data Information Provider must expose a DCAT-AP 3.0 compliant metadata catalogue endpoint with support for OAI-PMH or CSW harvesting. Each dataset entry must include `dcat:DataService` blocks with `dcat:endpointURL` and `dcat:endpointDescription` pointing to the MCP server, the underlying REST API, and any bulk download endpoints.

**Recommended change:** Publish a DCAT-AP catalogue endpoint (e.g., `https://mcp.mospi.gov.in/catalogue`) that describes all 19 datasets with their temporal coverage, spatial coverage, keywords, and available access interfaces. The `dcat:DataService` entries should point to both the MCP server and the underlying `api.mospi.gov.in` REST endpoints, with `dcat:endpointDescription` linking to the swagger YAML files already in the repository.

**Impact:** This makes MoSPI's datasets discoverable by any DCAT-AP catalogue harvester worldwide - the same mechanism used by the European Open Data Portal. It also resolves the discoverability problem without requiring changes to the MCP tool workflow itself.

### 1.2 Publish JSON Schema + Frictionless Data Package Descriptors

**Current state:** The swagger YAML files in the repository define *API parameters* (filter codes, valid values), but there is no schema describing the *data itself* - what fields are in the response, their types, constraints, or semantics. Thurro's report documents that all values are returned as strings (5.2.4), units are inconsistent (5.2.5), and null handling is undocumented (5.2.3).

**What the spec requires:** Every data product must include a JSON Schema (2020-12) and a Frictionless Data Package descriptor (`datapackage.json`) specifying data types, constraints, temporal coverage, update frequency, and units of measurement. All field names must be in `snake_case`.

**Recommended change:** For each of the 19 datasets, create a `datapackage.json` that formally declares the response schema. For example, the PLFS response should declare `value` as `type: number` (not string), `unit` as a controlled vocabulary field, and primary keys. This is a documentation exercise that can be generated semi-automatically from the existing swagger files and sample API responses.

**Why this matters beyond documentation:** A Frictionless descriptor allows a consuming system to programmatically determine fitness-for-use *before* making an API call. It also directly addresses three of Thurro's critical/high findings: string-typed values (5.2.4), missing unit standardisation (5.2.5), and undocumented null semantics (5.2.3), by creating a formal contract that the API responses should conform to.

### 1.3 Add a Data Quality Scorecard

**Current state:** There is no quality metadata. The Thurro report documents concrete quality issues - type fidelity is low (all values are strings), null semantics are undocumented, identifier consistency across datasets is zero, and precision is inappropriate (GDP with 15-digit decimal values).

**What the spec requires:** Each Data Passport must include a quality section with three tiers of metrics: Core (completeness, accuracy, uniqueness, validity), Structural Integrity (type fidelity, null semantics, unit standardisation, precision appropriateness, identifier consistency, naming convention compliance), and Operational (timeliness, update adherence, release calendar, revision transparency, error diagnostics, pagination transparency, methodology reference, data dictionary).

**Recommended change:** Compute and publish a Data Quality Scorecard for each dataset. Based on Thurro's findings, the current scores would be revealing - and that transparency is the point. For example:

- **Type fidelity:** ~0.0 (all values returned as strings)
- **Null semantics:** false (undocumented)
- **Identifier consistency:** ~0.0 (state codes differ across PLFS, CPI, NAS)
- **Naming convention compliance:** ~0.5 (inconsistent parameter names across datasets)
- **Error diagnostics:** none (generic empty responses)
- **Pagination transparency:** false (silent truncation at 10 records)
- **Data dictionary:** false
- **Release calendar:** false

Publishing these scores is not an admission of failure - it's a transparency mechanism that tells consumers exactly what to expect, and creates a measurable improvement trajectory.

### 1.4 Sign Data with Provenance Credentials

**Current state:** No provenance metadata. The Thurro report notes that NAS returns multiple revision vintages with no way to identify which is authoritative (5.2.1), and CPI has an undocumented "status" field for provisional/final.

**What the spec requires:** Every dataset version must be signed with a SHA-256 hash and accompanied by a W3C Verifiable Credential attesting to its provenance, including lineage claims.

**Recommended change:** At minimum, add a `revision_status` field to all API responses indicating whether the value is provisional, revised, or final. For NAS specifically, add an `is_latest` boolean or a `revision_rank` integer so consumers don't need to hardcode revision priority logic. Longer-term, implement W3C VC-based provenance signing for dataset snapshots.

---

## 2. Connect Layer Alignment

The Connect layer resolves entities and aligns meanings across data sources. This is where MoSPI's most painful interoperability failures live.

### 2.1 Adopt Standard Geographic Identifiers

**Current state:** This is the single most impactful gap. PLFS uses state_code 1–38 (with 99 for All India), CPI uses state_code 1–36 (with 99 for All India), and neither maps to Census codes, ISO 3166-2:IN codes, or LGD codes. Thurro had to build a manual mapping table to correlate unemployment with inflation by state.

**What the spec requires:** The Connect layer mandates LGD codes (or equivalent national registry) for sub-national geographic entities, with entity resolution mappings published as JSON-LD documents.

**Recommended change:** This is a two-phase effort:

*Phase 1 (non-breaking):* Add an LGD code field to all API responses alongside the existing dataset-specific state codes. The `step3_get_metadata()` response for each dataset's state filter should include a `lgd_code` or `census_code` alongside the internal code. This doesn't break existing consumers.

*Phase 2:* Publish an entity resolution mapping endpoint (e.g., `/crosswalk/geography`) that returns the full mapping between PLFS codes, CPI codes, LGD codes, and Census 2011 codes. This is the "crosswalk endpoint" recommended in Thurro's report, formalised as a Connect layer entity resolution mapping.

### 2.2 Standardise Time Period Identifiers

**Current state:** PLFS uses financial years (2023–24), CPI uses calendar years (2024), and NAS uses financial years (2024–25) but formatted differently. The `year` parameter has different semantics across datasets.

**What the spec requires:** Entity resolution for temporal identifiers, with a canonical format.

**Recommended change:** Accept both financial year (`2023-24`) and calendar year (`2024`) formats across all dataset endpoints, with the server performing internal mapping. Return a structured temporal object in responses:

```json
{
  "temporal": {
    "financial_year": "2023-24",
    "calendar_year_start": 2023,
    "calendar_year_end": 2024,
    "format": "financial_year_apr_mar"
  }
}
```

### 2.3 Add Semantic Context Documents (JSON-LD)

**Current state:** Field names like `indicator_code`, `gender_code`, `approach_code` are opaque integers with no semantic meaning. The Thurro report notes this makes it impossible to construct queries without first calling the metadata endpoint (5.1.2, 5.5.1).

**What the spec requires:** Every field must be mapped to a URI from a recognised vocabulary via JSON-LD context documents. Fields should use SDMX concept identifiers for statistical indicators, ISO 3166 for geography, etc.

**Recommended change:** Publish a JSON-LD context document for each dataset that maps internal codes to standard vocabularies. For example:

```json
{
  "@context": {
    "indicator_code_3": {
      "@id": "sdmx:UNEMPLOYMENT_RATE",
      "description": "Unemployment rate (usual status, ps+ss)"
    },
    "state_code_25": {
      "@id": "lgd:state:33",
      "sameAs": "iso3166-2:IN-TN",
      "label": "Tamil Nadu"
    }
  }
}
```

This doesn't require changing the API's internal codes - it provides a semantic "translation layer" that enables AI agents and automated systems to understand the meaning without calling `step3_get_metadata()` every time.

---

## 3. Enable Layer Alignment

The Enable layer transforms data into AI-ready representations. Since MoSPI chose MCP as its protocol, this is where the MCP server design itself needs the most attention.

### 3.1 Redesign MCP Tools for Direct Query Access

**Current state:** The mandatory four-step sequential workflow (discover → indicators → metadata → data) means every query requires four round-trip tool calls. The Thurro report benchmarks this against FRED, World Bank, and Eurostat - all of which support direct single-call queries.

**What the spec requires:** MCP tool definitions should have "clear names, descriptions, and parameter schemas" that allow direct invocation. The Data Unlock MCP tool example shows a single tool (`query_trade_statistics`) with self-documenting parameters including enums and descriptions.

**Recommended change:** Add a `step5_direct_query()` tool (or per-dataset query tools like `query_plfs`, `query_cpi`, `query_nas`) that accepts human-readable parameters and performs the indicator/metadata resolution internally. For example:

```json
{
  "name": "query_plfs",
  "description": "Query Periodic Labour Force Survey data",
  "parameters": {
    "indicator": {
      "type": "string",
      "enum": ["unemployment_rate", "lfpr", "wpr", "avg_wage_regular", "avg_wage_casual", "avg_wage_self_employed"],
      "description": "The indicator to query"
    },
    "state": {
      "type": "string",
      "description": "State name or LGD code (e.g., 'Tamil Nadu' or 'LGD:33')"
    },
    "year": {
      "type": "string",
      "description": "Financial year (e.g., '2023-24') or calendar year (e.g., '2024')"
    },
    "sector": {
      "type": "string",
      "enum": ["rural", "urban", "combined"],
      "default": "combined"
    }
  }
}
```

The existing four-step workflow can remain available for advanced users who need to explore the full parameter space, but the direct query tool would handle the common case in a single call.

### 3.2 Add `source_ref` and Provenance to MCP Responses

**Current state:** MCP responses return raw data arrays with no provenance metadata. An AI agent consuming the data cannot cite where it came from or verify its freshness.

**What the spec requires:** MCP tool responses must include `source_ref` fields pointing to Data Passport URIs. The Apply layer further requires that every factual claim in an AI response must be traceable to a specific data source.

**Recommended change:** Every `step4_get_data()` (and any new direct query tool) response should include:

```json
{
  "data": [...],
  "source_ref": "urn:mospi:dp:plfs-v2",
  "freshness": "2024-03-31",
  "revision_status": "final",
  "methodology_url": "https://mospi.gov.in/publication/plfs-methodology-2024",
  "data_passport_url": "https://mcp.mospi.gov.in/passport/plfs"
}
```

This enables downstream AI applications to provide grounded, citable responses - which is the entire point of exposing data through MCP rather than a traditional API.

### 3.3 Implement Informative Error Responses

**Current state:** When a query returns no data, the API returns an empty array with "No Data Found" - no distinction between invalid filters, unpublished data, wrong filter values, or server errors (Thurro 5.5.2).

**What the spec requires:** The Data Quality Scorecard's "Error Diagnostics" metric requires error responses that distinguish between invalid requests, not-yet-published data, empty result sets, and server errors.

**Recommended change:** Return structured error objects:

```json
{
  "status": "no_results",
  "reason": "invalid_filter_combination",
  "details": "state_code 25 is valid for PLFS but indicator_code 9 does not exist. Valid indicator codes are 1-8.",
  "suggestion": "Did you mean indicator_code=3 (Unemployment Rate)?"
}
```

For an MCP server specifically designed for AI consumption, this is high-impact: it prevents the LLM from incorrectly concluding that data doesn't exist when the real problem is a parameter error.

### 3.4 Expose an OpenAPI Specification for the Underlying REST API

**Current state:** The swagger YAML files exist in the repository but are used only for internal parameter validation. There is no public OpenAPI endpoint.

**What the spec requires:** The Enable layer requires OpenAPI 3.1 specifications for REST interfaces. The Prepare layer's new DCAT DataService section requires `dcat:endpointDescription` pointing to a machine-readable interface description.

**Recommended change:** Consolidate the per-dataset swagger files into a single OpenAPI 3.1 specification and serve it at a public endpoint (e.g., `https://mcp.mospi.gov.in/openapi.json`). This serves double duty: it satisfies the DCAT DataService requirement, and it gives non-MCP consumers (traditional REST clients, data pipelines) a self-documenting interface.

### 3.5 Add Pagination Transparency

**Current state:** The default limit is 10 records with silent truncation. The maximum limit is undocumented. Thurro discovered through experimentation that limit=300 works, but there is no guarantee of stability.

**What the spec requires:** The Pagination Transparency metric requires that the API documents its result size limits and clearly signals when results are truncated.

**Recommended change:** Include pagination metadata in every response:

```json
{
  "data": [...],
  "pagination": {
    "total_records": 210,
    "returned_records": 210,
    "limit": 300,
    "offset": 0,
    "is_truncated": false,
    "max_limit": 1000
  }
}
```

Change the default limit to 100 (as Thurro recommends) and document the maximum.

---

## 4. Cross-Cutting: Data Passport

The Data Passport is the single metadata envelope that ties all layers together. MoSPI currently has no equivalent.

### 4.1 Publish a Data Passport for Each Dataset

**Recommended change:** Create a Data Passport endpoint (e.g., `https://mcp.mospi.gov.in/passport/{dataset}`) that returns a JSON-LD document combining:

- **Identity:** Publisher (MoSPI/NSO), version, temporal and spatial coverage
- **Discovery:** DCAT-AP metadata (links to the catalogue entry)
- **Structure:** Reference to the JSON Schema / Frictionless descriptor
- **Quality:** The Data Quality Scorecard
- **Provenance:** Revision status, last updated timestamp, lineage
- **Policy:** Access terms (currently open/public, but formalising this in ODRL future-proofs it)
- **Semantics:** Links to JSON-LD context documents
- **AI Readiness:** MCP server endpoint, available tools, supported formats

This is not a large engineering effort - most of the information already exists in the swagger files, the `step1` and `step2` tool responses, and MoSPI's published methodology documents. It just needs to be assembled into the standard envelope.

---

## 5. MCP-Specific Design Improvements

These are MCP best practices related improvements to the MCP server.

### 5.1 Add a Data Dictionary Tool

**Current state:** No formal definitions for any indicator. The Thurro report documents serious interpretation risks - terms like "PS+SS" (principal status plus subsidiary status), "CWS" (current weekly status), "invested capital," and "gross value of addition to fixed capital" have specific statistical meanings that are not documented anywhere in the API.

**Recommended change:** Add a `get_data_dictionary` MCP tool that returns, for each indicator: formal definition, unit of measurement, methodology reference URL, frequency of publication, known caveats, and date of last update.

### 5.2 Fix the `frequency_code` Semantics in PLFS

**Current state:** The `frequency_code` parameter in PLFS is deeply misleading. Code 1 (labelled "annual") actually selects a set of 8 indicators available at annual frequency. Code 2 ("quarterly") selects a different set of 4 indicators from quarterly bulletins. Code 3 ("monthly") selects 3 indicators only available from 2025 onwards. It does not control time granularity - it selects indicator sets.

**Recommended change:** Rename to `indicator_set` with values like `annual_report` (8 indicators), `quarterly_bulletin` (4 indicators), `monthly_bulletin` (3 indicators). Update the tool descriptions accordingly. This is a small change that prevents systematic misinterpretation by both humans and AI agents.

### 5.3 Support Multi-Indicator Queries for ASI

**Current state:** Each of ASI's 57 indicators must be fetched individually. Constructing a complete industry profile for a single NIC group and year requires 57 separate API calls.

**Recommended change:** Allow `indicator_code` to accept a comma-separated list or an `all` keyword in ASI queries, returning a wide-format response with one row per NIC group and one column per indicator. This would reduce a complete industry profile from 57 calls to 1.

---

## Summary: Priority Order

Mapping the improvements against both Thurro's severity ratings and Data Unlock compliance, the highest-impact changes in order are:

1. **Standard geographic identifiers + crosswalk endpoint** - resolves the #1 cross-dataset barrier (Thurro Critical, Connect layer)
2. **Direct query MCP tool** - eliminates the 4-call overhead for common queries (Thurro Critical, Enable layer)
3. **Informative error responses** - prevents AI agents from reaching wrong conclusions (Thurro High, Enable layer)
4. **Data dictionary tool** - enables correct interpretation of indicators (Thurro Critical, Prepare layer)
5. **Numeric types + unit metadata in responses** - fixes structural integrity (Thurro High, Prepare layer)
6. **Pagination transparency** - prevents silent data truncation (Thurro Medium, Enable layer)
7. **DCAT-AP catalogue with DataService entries** - makes datasets discoverable (Prepare layer)
8. **JSON-LD semantic context documents** - enables automated understanding (Connect layer)
9. **`source_ref` in MCP responses** - enables grounded AI responses (Enable + Apply layers)
10. **Data Passport per dataset** - ties everything together (cross-cutting)
