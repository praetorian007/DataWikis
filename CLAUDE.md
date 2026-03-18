# DataWikis

Wiki-style documentation for Water Corporation covering data, analytics, and governance domains.

## Repository structure

```
lifecycles/       — Lifecycle frameworks (BI, Data Engineering, Data Science, Data Governance)
platform/         — Platform and architecture docs (Medallion, Databricks, EDAP access model, enterprise data models)
governance/       — Governance policies, roles, tagging strategy, domain governance
specifications/   — Technical specifications and framework requirements
```

## Content conventions

- Documents are authored by Mark Shaw, Principal Data Architect
- Standard header format: `# Title`, then `**Mark Shaw** | Principal Data Architect`, then `---`
- Written in Australian English (organisation, centralised, etc.)
- Markdown with tables, diagrams described in text, and structured headings

## Domain context

- **Water Corporation** — Western Australian water utility
- **EDAP** — Enterprise Data & Analytics Platform, built on Databricks lakehouse with Unity Catalog
- **Medallion Architecture** — Bronze/Silver/Gold layers with zone decomposition (Landing, Raw, Processed, Protected, Base, Enriched, Exploratory, Dimensional, Sandbox, Quarantine)
- **Unity Catalog** — Databricks governance layer for access control, lineage, and metadata
- **Alation** — Enterprise data discovery and cataloguing tool
- **Seven data domains** — Water Corporation organises data across seven business domains
- **Regulatory context** — SOCI Act 2018, PRIS Act 2024, WAICP, State Records Act 2000, Essential Eight

## When editing or adding documents

- Maintain consistency with existing document style and structure
- Use domain-specific terminology as established in existing docs
- Keep content tailored to Water Corporation's context and infrastructure
