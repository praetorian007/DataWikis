# Databricks End-to-End Data Platform: From Ingestion to AI-Powered BI

**Mark Shaw** | Principal Data Architect

---

## 1. Platform Overview

The Databricks Data Intelligence Platform has consolidated around three pillars — **Lakeflow** (ingestion, transformation, orchestration), **Unity Catalog** (governance), and **AI/BI** (consumption and self-service analytics). The platform now operates as a fully integrated stack where data flows from source systems through a medallion architecture (Bronze → Silver → Gold) and surfaces to business users via AI/BI Dashboards and Genie spaces — all governed by Unity Catalog and executed on serverless compute secured by Lakeguard.

This document traces that end-to-end journey, layer by layer, through the lens of currently GA and Public Preview features.

---

## 2. Ingestion: Lakeflow Connect

### 2.1 What It Is

Lakeflow Connect is Databricks' native, fully-managed ingestion service. It provides point-and-click (or API/CLI-driven) connectors for SaaS applications, databases, cloud storage, message buses, and file sources. All ingestion pipelines are governed by Unity Catalog, powered by serverless compute, and built on Lakeflow Spark Declarative Pipelines under the hood.

### 2.2 Connector Landscape (GA and Public Preview)

**Fully-managed SaaS connectors (GA):** Salesforce Sales Cloud, Workday, Google Analytics 4, ServiceNow, SharePoint, Oracle NetSuite.

**Fully-managed database connectors (GA/Preview):** Microsoft SQL Server (Azure SQL, Amazon RDS, on-premises via ExpressRoute/Direct Connect), PostgreSQL (Public Preview as of Dec 2025), MySQL (Public Preview as of Dec 2025). Connectors for SFTP, IBM DB2, MongoDB, Amazon DynamoDB, and Confluence are in various preview/development stages.

**Standard (customisable) connectors:** Auto Loader for cloud storage (S3, ADLS Gen2, GCS), Apache Kafka, Amazon Kinesis, Google Pub/Sub, Azure Event Hubs, and Apache Pulsar. These can run in either Structured Streaming (full customisation) or Lakeflow Spark Declarative Pipelines (managed experience).

**Custom connectors:** Any language supported by Databricks (Python, Java, Scala), plus open-source libraries such as dlt (data load tool), Airbyte, and Debezium.

**Partner ingestion:** Fivetran, Qlik, Informatica, and others via Partner Connect.

### 2.3 Key Architecture Concepts

**Incremental ingestion:** Lakeflow Connect uses incremental reads and writes by default. For database connectors, Change Data Capture (CDC) technology (from the Arcion acquisition) enables efficient, low-impact replication without full table scans.

**Ingestion gateway:** For database connectors, a dedicated gateway runs as a continuous task that captures change events. The ingestion pipeline itself runs on a configurable schedule, consuming those changes.

**Cursor-based recovery:** On failure, connectors store the last cursor position and retry with exponential backoff. When credentials expire or an external service changes, the connector picks up from the stored position on the next successful run.

**Infrastructure as Code:** Ingestion pipelines can be deployed via **Databricks Asset Bundles (DABs)** — enabling source control, code review, CI/CD, and multi-environment deployment (dev → staging → prod). Terraform support is also available, and the two approaches can be used complementarily. DABs are the primary IaC deployment model for all Databricks assets, including jobs, pipelines, notebooks, ML models, and dashboard configurations, providing a single declarative YAML definition per project.

### 2.4 Landing Zone Pattern

Raw data from Lakeflow Connect lands directly into Delta tables in your Bronze catalog/schema. For file-based ingestion via Auto Loader, data typically lands in Unity Catalog Volumes (managed or external) before being read into streaming tables. The `cloud_files` format in Auto Loader supports schema inference, schema evolution, and the `VARIANT` column type for semi-structured data (JSON).

```sql
-- Example: Auto Loader ingestion into a Bronze streaming table
CREATE OR REFRESH STREAMING TABLE bronze_orders
AS SELECT * FROM cloud_files(
  '/volumes/landing/erp/orders/',
  'json',
  map('cloudFiles.inferColumnTypes', 'true')
);
```

---

## 3. Transformation: Lakeflow Spark Declarative Pipelines (SDP)

### 3.1 What SDP Is

Lakeflow Spark Declarative Pipelines (SDP) — the evolution of Delta Live Tables (DLT) — is a declarative framework for building batch and streaming ETL pipelines in SQL or Python. You declare *what* datasets should exist and *how* they relate; SDP handles dependency resolution, incremental processing, compute autoscaling, retry logic, and data quality enforcement.

SDP is also available as an open-source component in Apache Spark 4.1+ (Spark Declarative Pipelines), with the Databricks-managed version extending this with performance optimisations, serverless compute, and tight Unity Catalog integration.

### 3.2 Core Constructs

**Streaming Tables:** Append-only tables that process data incrementally. Ideal for Bronze ingestion and any workload where new records arrive continuously. Auto Loader sources feed directly into streaming tables.

**Materialized Views:** Tables that are recomputed (or incrementally refreshed where possible) based on a declarative query. Ideal for Silver and Gold aggregations where the full result set matters, not just new rows.

**Temporary Views:** Intermediate query results that exist only within the pipeline execution context.

### 3.3 The Medallion Architecture in SDP

**Bronze (Raw):** Streaming tables ingesting from Lakeflow Connect or Auto Loader. Data is stored as-is with append-only semantics. Schema enforcement is minimal — the goal is a faithful copy of the source.

```sql
-- Bronze: raw CDC events from SQL Server via Lakeflow Connect
CREATE OR REFRESH STREAMING TABLE bronze_customers
AS SELECT * FROM STREAM read_files(
  '/volumes/landing/sqlserver/customers_cdc/'
);
```

**Silver (Cleaned/Conformed):** Materialized views or streaming tables that apply data quality expectations, deduplication, type casting, and business key resolution. CDC processing uses the `AUTO CDC INTO` syntax (or `APPLY CHANGES INTO` in Python) to produce SCD Type 1 or Type 2 outputs.

```sql
-- Silver: SCD Type 2 from CDC events
CREATE OR REFRESH STREAMING TABLE silver_customers;

AUTO CDC INTO silver_customers
FROM STREAM bronze_customers
KEYS (customer_id)
SEQUENCE BY _commit_timestamp
STORED AS SCD TYPE 2;
```

**Gold (Business-Ready):** Materialized views producing dimensional models, aggregations, and KPI tables optimised for BI consumption. These tables are the primary source for AI/BI Dashboards and Genie spaces.

```sql
-- Gold: daily revenue aggregation
CREATE OR REFRESH MATERIALIZED VIEW gold_daily_revenue
AS SELECT
  date_trunc('day', order_date) AS order_day,
  region,
  SUM(amount) AS total_revenue,
  COUNT(DISTINCT customer_id) AS unique_customers
FROM silver_orders
GROUP BY ALL;
```

### 3.4 Data Quality Expectations

SDP supports inline data quality rules using `EXPECT`, `EXPECT OR DROP`, and `EXPECT OR FAIL` syntax. These are evaluated at pipeline runtime and surfaced in the pipeline UI and event log.

```sql
CREATE OR REFRESH STREAMING TABLE silver_orders (
  CONSTRAINT valid_amount EXPECT (amount > 0) ON VIOLATION DROP ROW,
  CONSTRAINT valid_date EXPECT (order_date IS NOT NULL) ON VIOLATION FAIL UPDATE
)
AS SELECT * FROM STREAM bronze_orders;
```

Expectation metrics (rows passed, failed, dropped) are written to the pipeline event log as Delta tables, enabling downstream alerting and dashboarding.

### 3.5 Pipeline Execution Modes

**Triggered (batch):** Pipeline runs on a schedule or on-demand, processes all available data, and terminates. Cost-efficient for workloads that tolerate some latency.

**Continuous (streaming):** Pipeline runs continuously with low-latency processing. Suitable for real-time use cases.

**Serverless:** SDP pipelines run on serverless compute by default (where available), with automatic scaling, warm pools for fast start-up, and pay-for-work-done billing.

---

## 4. Unity Catalog: Governance Across the Pipeline

### 4.1 The Three-Level Namespace

All data assets are addressed as `catalog.schema.object`. The catalog is the primary unit of data isolation — commonly aligned to environments (e.g., `dev_bronze`, `prod_silver`), organisational units, or data domains. Catalogs can be organised by environment, by domain, by medallion layer, or by a combination of these axes. See the companion **EDAP Access Model** document for Water Corporation's specific layer-based catalog approach (`<env>_<layer>`, e.g. `prod_bronze`, `prod_silver`, `prod_gold`).

**Object types governed:** Tables (managed and external), views, materialized views, streaming tables, volumes (structured and unstructured files), ML models, functions (UDFs), connections, metrics, and AI agents. **Unity Catalog Volumes** are the managed replacement for the legacy DBFS root and mount points. Volumes provide governed, access-controlled storage for unstructured data (images, PDFs, audio, video, model artefacts) and are essential for applying the same governance framework to file-based assets as to tabular data.

### 4.2 Access Control

Unity Catalog provides a hierarchical, ANSI SQL-based privilege model. Key mechanisms include:

- **ACL grants:** `GRANT SELECT ON TABLE catalog.schema.table TO group_name`
- **Row filters and column masks:** Fine-grained access control applied declaratively at the table level.
- **Attribute-Based Access Control (ABAC):** Now in Public Preview at account level. Define policies based on governed tags applied to data assets. ABAC policies are enforced across all workspaces attached to the metastore. Column mask functions in ABAC policies now support automatic type casting.

### 4.3 Governed Tags and Classification

Tags are first-class metadata objects in Unity Catalog. You can apply tags at catalog, schema, table, and column levels. Tags integrate with ABAC policies for dynamic access control. Use cases include sensitivity classification, regulatory alignment, data domain assignment, lifecycle management, and AI/ML asset governance. The companion **EDAP Tagging Strategy** defines Water Corporation's four-layer tag taxonomy — including `waicp_classification`, `pi_contained`, `pi_type`, `pi_lawful_basis`, `regulatory_scope`, `sensitivity_type`, `access_model`, `masking_required`, `data_domain`, `soci_critical`, `quality_tier`, `model_risk_tier`, and `ai_governance_level` — with allowed values, inheritance rules, and change management procedures.

### 4.4 Lineage

Unity Catalog automatically captures column-level lineage across all queries executed on Databricks compute (notebooks, jobs, SQL warehouses, SDP pipelines). Lineage is visualised in Catalog Explorer and is queryable via system tables (`system.access.column_lineage`, `system.access.table_lineage`).

### 4.5 Data Discovery

**Catalog Explorer:** Browse, search, and manage all data and AI assets. The Insights tab shows the most frequent recent queries and users for any table.

**AI-powered documentation:** Unity Catalog can auto-generate table and column descriptions using AI, reducing the manual metadata curation burden.

**UC Metrics (Public Preview):** Business metrics defined as first-class catalog objects. Define a metric once in Unity Catalog and consume it across AI/BI Dashboards, Genie spaces, notebooks, SQL, and Lakeflow Jobs. Upcoming integrations will extend to Tableau, Hex, Sigma, ThoughtSpot, Omni, and observability tools like Anomalo and Monte Carlo.

### 4.6 Lakehouse Federation

Lakehouse Federation enables Unity Catalog to query external data sources — SQL Server, PostgreSQL, MySQL, Snowflake, BigQuery, Amazon Redshift, and others — without copying data into the lakehouse. Federation uses **connections** (Unity Catalog objects that store credentials and endpoint details) and **foreign catalogs** (read-only catalog mappings that expose external schemas and tables within the three-level namespace).

Federated tables appear in Catalog Explorer, participate in lineage tracking, and can be governed with tags and access controls like any Unity Catalog asset. This enables use cases such as joining lakehouse Gold tables with operational system data for real-time enrichment, querying data warehouse tables during migration without full replication, and building cross-platform analytical views — all while maintaining a single governance plane.

See the companion **Domain Governance Across Systems** document for the governance process applied to federated data sources, including the `_federated` naming suffix convention defined in the **EDAP Tagging Strategy**.

### 4.7 Iceberg Interoperability

Unity Catalog now provides full support for the Iceberg REST Catalog API — external engines can both read (GA) and write (Public Preview) to UC-managed Iceberg tables. Iceberg managed tables deliver liquid clustering, predictive optimisation, and full integration with Databricks and external engines (Trino, Snowflake, Amazon EMR). Iceberg catalog federation allows querying Iceberg tables in AWS Glue, Hive Metastore, and Snowflake without copying data.

### 4.8 Delta Sharing

Delta Sharing provides open, cross-platform data sharing with governance. Streaming tables and materialized views can now be shared (GA as of mid-2025). ABAC-secured tables can be included in shares. External Iceberg clients (Snowflake, Trino, Flink, Spark) can query shared Delta tables with zero-copy access.

For the six-step governance process applied to external Delta Sharing at Water Corporation, see the companion **Domain Governance Across Systems** document.

### 4.9 Clean Rooms

Databricks Clean Rooms (GA) enable privacy-preserving collaboration on shared data without exposing raw records. Participants define their own tables and the approved computations (aggregations, joins, ML training) that can run against them — only the computation results are returned, never the underlying data. Clean Rooms are governed by Unity Catalog, use serverless compute, and support both SQL and notebook-based collaborations. This capability is relevant for regulatory scenarios (sharing anonymised operational data with government agencies), cross-organisation analytics (joint analysis with partner utilities or suppliers), and any use case where data must be analysed collaboratively without physical data movement or exposure.

### 4.10 Predictive Optimization

Predictive Optimization (GA) automatically manages table maintenance operations — `OPTIMIZE` (file compaction and liquid clustering), `VACUUM` (removal of unreferenced files), and `ANALYZE TABLE` (statistics collection) — for managed tables in Unity Catalog. The service monitors table usage patterns and file layout health, scheduling maintenance operations when they will deliver the most benefit. This eliminates the operational overhead of manual table maintenance, reduces storage costs through timely cleanup, and improves query performance through proactive optimisation — all without requiring user intervention or scheduling.

### 4.11 Marketplace

Databricks Marketplace provides a discovery and distribution platform for shared data products, notebooks, ML models, Databricks Apps, and solution accelerators. Providers can publish assets for open or restricted access; consumers can discover and install them directly into their Unity Catalog. For enterprise use, Marketplace supports private exchanges — enabling internal teams to publish governed data products for discovery and consumption by other teams within the organisation, complementing the data product lifecycle described in the companion **Data Engineering Lifecycle**.

### 4.12 System Tables

Databricks exposes operational metadata as queryable Delta tables under the `system` catalog, covering audit logs, billing, compute usage, lineage, job runs, pipeline metrics, and network access events. These are essential for building governance dashboards, cost attribution, and compliance reporting.

---

## 5. Compute: Serverless and Lakeguard

### 5.1 Serverless Compute

Serverless compute is GA for notebooks, jobs, SDP pipelines, and SQL warehouses across AWS and Azure. Key characteristics: seconds-level start-up from warm pools, automatic infrastructure scaling, pay-for-work-done billing, and no instance type selection required.

Serverless compute requires Unity Catalog for data access. Workloads must be compatible with Databricks Runtime 14.3+ and standard access mode.

New Databricks accounts created after December 2025 default to Unity Catalog-only — DBFS root, mounts, Hive Metastore, and no-isolation shared compute are disabled.

### 5.2 Lakeguard

Lakeguard is the isolation and data filtering layer that underpins serverless and shared compute. It provides:

- **Spark Connect decoupling:** Client applications run in separate processes from the Spark driver, preventing JVM-level data leakage and enforcing row/column-level filters correctly even against over-fetching.
- **Container sandboxing:** Each client application (and each user in shared compute) runs in its own isolated container, preventing cross-user data access.
- **UDF sandboxing:** User-defined functions execute in sandboxed environments with isolated egress network traffic, preventing unauthorised external access.

Lakeguard is what makes fine-grained access control (ABAC, row filters, column masks) enforceable at runtime, including for distributed ML workloads (Spark MLlib on serverless is now Public Preview).

### 5.3 Cost Management

Effective cost management on Databricks centres on three capabilities:

- **Serverless billing:** Serverless compute uses pay-for-work-done pricing, eliminating idle cluster costs. Understanding the billing model for serverless SQL warehouses, jobs, SDP pipelines, and Model Serving endpoints is essential for forecasting.
- **System tables for cost attribution:** The `system.billing.usage` and related system tables enable granular cost attribution by workspace, job, pipeline, user, and tag. These tables are the foundation for chargeback and showback models across domains.
- **Cluster policies:** For non-serverless compute, cluster policies enforce guardrails on instance types, autoscaling ranges, spot/on-demand ratios, and automatic termination. Policies should be defined per team or workload type to prevent cost overruns.

---

## 6. Orchestration: Lakeflow Jobs

### 6.1 Core Concepts

Lakeflow Jobs (formerly Databricks Workflows) is the native orchestrator for all workloads on the Data Intelligence Platform. It coordinates ingestion, transformation, notebooks, SQL queries, ML training, model deployment, dashboard refreshes, and more.

A **job** is a DAG of **tasks** with **triggers**. Tasks can be notebook, Python script, SQL, SDP pipeline, Lakeflow Connect ingestion pipeline, dbt, JAR, or AI/BI dashboard refresh operations. Control flow supports branching (`if/else`), looping (`for each`), retries, and task dependencies.

### 6.2 Triggers

- **Scheduled:** Cron-based or fixed-interval.
- **File arrival:** Trigger when new files land in cloud storage.
- **Table update:** Trigger when upstream Unity Catalog tables are updated.
- **Continuous:** Run jobs in continuous mode for always-on workloads.
- **On-demand / API:** Trigger via REST API, CLI, Terraform, or Databricks Asset Bundles.

### 6.3 Monitoring and Observability

Job run history, task-level execution times, and status are visible in the UI. System tables (`system.lakeflow.jobs`, `system.lakeflow.job_run_timeline`, `system.lakeflow.job_task_run_timeline`) provide queryable metrics including setup, queue, execution, and cleanup durations. Notifications can be sent via email, Slack, Microsoft Teams, or custom webhooks.

### 6.4 Integration with Lakeflow Connect

Ingestion pipelines created via Lakeflow Connect are automatically orchestrated as tasks within Lakeflow Jobs. You can add transformation, quality check, and downstream refresh tasks to the same job — creating a single end-to-end pipeline from source to Gold.

---

## 7. Data Quality Monitoring

### 7.1 In-Pipeline: SDP Expectations

As described in Section 3.4, data quality expectations are enforced inline during pipeline execution. This is your first line of defence.

### 7.2 Post-Pipeline: Data Quality Monitoring (formerly Lakehouse Monitoring)

Data Quality Monitoring is a serverless, Unity Catalog-integrated service that tracks the health of your tables over time:

- **Anomaly detection:** One-click enablement at schema level. Automatically monitors table freshness (when was the table last updated?) and completeness (are we getting the expected number of rows?) by analysing historical commit patterns and building per-table models.
- **Data profiling:** Computes summary statistics (mean, min/max, null percentages, distribution metrics) for any Delta table in Unity Catalog. Supports time-series, snapshot, and inference analysis modes.
- **Custom metrics:** Define business-specific metrics that are computed alongside built-in ones.
- **Dashboard and alerting:** Auto-generates a fully customisable dashboard. Metrics are stored as Delta tables, enabling ad-hoc SQL queries and Databricks SQL alerts on threshold violations, distribution drift, or freshness anomalies.

---

## 8. The BI Layer: AI/BI Dashboards and Genie

### 8.1 AI/BI Dashboards (GA)

AI/BI Dashboards are the native BI experience in Databricks, tightly integrated with Unity Catalog governance and SQL Warehouses.

**Key capabilities (current as at early 2026):**

- **Visualisation types:** Bar, line, area, scatter, pie, heatmap, funnel, waterfall, histogram, pivot table (with drill-through and hierarchies), point map, counter, and table widgets.
- **Cross-filtering:** Click a data point in one widget to filter others.
- **Parameters and filters:** Global and page-level filters, date range parameters with "All" support, multi-select, and DD-MM-YYYY date formatting.
- **Embedding:** GA for internal users; secure embedding for external users (customers, partners) via OAuth and service principal authentication — no Databricks account required for viewers.
- **Subscriptions:** Scheduled snapshots delivered to email, Slack channels, or Microsoft Teams channels (Public Preview), with PNG image preview, direct link, and PDF attachment.
- **Genie Code for authoring (Public Preview):** Use natural language prompts to automate multi-step dashboard creation — datasets, visualisations, layouts, and filters.
- **Custom fonts and workspace-level colour palettes.**

### 8.2 AI/BI Genie (GA)

Genie is the conversational analytics layer. Business users ask questions in natural language and receive answers as text summaries, tables, and visualisations.

**Architecture:** Each Genie space is backed by Unity Catalog tables and/or metric views. Analysts curate the space by adding tables, providing instructions, sample SQL, and example questions. Genie generates SQL behind the scenes, runs it against a SQL Warehouse, and returns results — all governed by the user's Unity Catalog permissions.

**Key capabilities:**

- **Companion spaces:** Every AI/BI Dashboard automatically creates a linked Genie space, allowing users to go beyond the predefined dashboard views.
- **Metric views:** Tables and metric views can be added to spaces and analysed together. Metric views provide semantic metadata (display names, formats) that Genie uses to produce clearer answers.
- **Research Agent (Beta):** Extends Genie with multi-step reasoning and hypothesis testing for complex analytical questions.
- **Agent mode (Beta):** Parallel query execution and inline thinking traces for faster, more transparent results.
- **Thinking steps:** Each response shows how the prompt was interpreted and which tables/SQL were used.
- **Benchmarks:** Space editors can evaluate Genie's answer accuracy with a built-in benchmarking framework.
- **Embedded credentials:** Genie queries run using the publishing user's warehouse permissions when dashboards are published with embedded credentials — including in externally embedded dashboards.
- **Genie API (GA):** `CreateSpace`, `UpdateSpace`, `GetSpace`, `ListSpaces`, `TrashSpace` — enabling integration into Slack, Microsoft Teams, or custom applications.
- **Automatic JOIN relationships:** The API now auto-creates JOIN relationships based on primary and foreign key metadata in Unity Catalog.

### 8.3 Databricks Apps

Databricks Apps enable teams to build and deploy custom data applications — using frameworks such as Streamlit, Dash, and Gradio — directly within the Databricks platform. Apps are governed by Unity Catalog, meaning they respect the same access controls, lineage, and tagging as any other platform asset. They provide a flexible consumption layer for interactive tools, operational dashboards, and domain-specific utilities that go beyond what standard BI dashboards offer.

### 8.4 Databricks One

Databricks One is a simplified portal for business users that surfaces AI/BI Dashboards, Genie spaces, and Databricks Apps without exposing the full technical workspace. It is freely available to all Databricks SQL customers. Global search now matches on dashboard content, page names, widget titles, and dataset queries.

### 8.5 Enabling Genie on Your Gold Tables

To get Genie working effectively against your Gold layer:

1. **Register Gold tables in Unity Catalog** with clear, descriptive table and column names. Apply comments/descriptions to all columns.
2. **Define metric views** for your core KPIs, centralising business logic so Genie (and dashboards) produce consistent answers.
3. **Add semantic metadata** to metric views (YAML spec v1.1+, DBR 17.2+) — display names, display formats, and descriptions that help Genie interpret the data correctly.
4. **Create a Genie space**, add your Gold tables and metric views, and provide curated instructions: common questions, sample SQL, domain-specific terminology, and any nuances about filters or date ranges.
5. **Benchmark and iterate.** Use the built-in benchmarking framework to evaluate answer quality and the feedback workflow to correct and regenerate incorrect responses.

---

## 9. Mosaic AI

Mosaic AI is Databricks' integrated AI and machine learning platform, tightly coupled with Unity Catalog governance and serverless compute.

### 9.1 Model Serving

Model Serving provides scalable, serverless endpoints for deploying custom models and accessing Foundation Model APIs (including models from Meta, Anthropic, Mistral, and others). Endpoints support real-time inference, batch scoring, and A/B testing with automatic traffic routing. All endpoints are governed by Unity Catalog and secured by Lakeguard.

### 9.2 Feature Engineering

Unity Catalog feature tables replace the legacy Feature Store. Features are defined as standard Delta tables with primary key and timestamp metadata, enabling point-in-time lookups for training and real-time serving. Feature tables are discoverable, governed, and lineage-tracked like any other Unity Catalog asset.

### 9.3 Vector Search

Vector Search provides a serverless, managed vector database for retrieval-augmented generation (RAG) workflows. It automatically syncs embeddings from Delta tables and supports filtered similarity search. Vector Search indexes are Unity Catalog objects, inheriting access controls and lineage.

### 9.4 AI Agent Framework

The Databricks AI Agent Framework provides tools for building, evaluating, and deploying compound AI systems (agents) that combine LLMs with tools such as Unity Catalog functions, retrieval, and code execution. Agents can be logged, versioned, and served using Model Serving endpoints. The Agent Evaluation framework enables systematic quality assessment before production deployment.

### 9.5 AI Gateway

AI Gateway (GA) provides a unified API layer for managing access to foundation models — both Databricks-hosted models and external providers (OpenAI, Anthropic, Azure OpenAI, Google, Cohere, and others). Key capabilities:

- **Provider routing and fallback:** Configure primary and fallback model providers per endpoint, enabling resilience and cost optimisation without application code changes.
- **Rate limiting and token budgets:** Enforce per-endpoint, per-user, or per-team rate limits and token consumption budgets — essential for controlling GenAI costs.
- **Guardrails:** Apply input/output guardrails (content filtering, PII detection, topic restrictions) at the gateway level, enforced consistently across all consuming applications.
- **Usage tracking and audit:** All requests and responses are logged, providing a single audit trail for compliance, cost attribution, and usage analysis. Usage data flows to system tables for integration with FinOps dashboards.
- **Centralised governance:** AI Gateway is the enforcement point for GenAI access policies — controlling which models are available to which teams, under what conditions, with what guardrails.

### 9.6 Online Tables

Online Tables (GA) provide low-latency, high-throughput key-value serving for ML feature lookups and real-time applications. An Online Table is a read-only, automatically synchronised copy of a Delta table optimised for point lookups by primary key. Online Tables are Unity Catalog objects, inheriting access controls, lineage, and tagging. They complete the feature serving story: features are defined and computed as standard Delta tables (Section 9.2), and Online Tables make those same features available for millisecond-latency serving at inference time — eliminating training-serving skew without requiring a separate feature serving infrastructure.

### 9.7 MLflow Integration

MLflow is deeply integrated into the Databricks platform for experiment tracking, model versioning, and the model registry (via Unity Catalog). Models registered in Unity Catalog carry lineage, governed tags, and access controls — enabling the same governance framework used for data to extend to AI assets.

---

## 10. Putting It All Together: End-to-End Data Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SOURCE SYSTEMS                               │
│  (SQL Server, Salesforce, Kafka, S3/ADLS/GCS, SharePoint, etc.)    │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                    Lakeflow Connect / Auto Loader
                    (Serverless, CDC, Incremental)
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     BRONZE (Raw / Landing)                          │
│  Streaming Tables in Unity Catalog                                  │
│  prod_bronze.source_system.table_name                               │
│  • Append-only, schema-on-read                                      │
│  • Lakeflow Connect manages cursor, retry, scheduling               │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                    SDP Pipeline (SQL/Python)
                    AUTO CDC INTO / Expectations
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     SILVER (Cleaned / Conformed)                    │
│  Streaming Tables + Materialized Views in Unity Catalog             │
│  prod_silver.domain.entity                                          │
│  • SCD Type 1/2, deduplication, type casting                        │
│  • DQ expectations (EXPECT OR DROP / FAIL)                          │
│  • Column-level lineage captured automatically                      │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                    SDP Pipeline (SQL/Python)
                    Aggregations, Dimensional Models
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     GOLD (Business-Ready)                           │
│  Materialized Views + Feature Tables in Unity Catalog               │
│  prod_gold.domain.metric_or_dim                                     │
│  • Dimensional models, KPI aggregations                             │
│  • UC Metrics defined for core business measures                    │
│  • Semantic metadata for Genie consumption                          │
│  • Feature tables for ML consumption                                │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
         ▼                     ▼                     ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐
│  BI Consumption  │  │  AI/ML Path      │  │  Sharing & Collab   │
│                 │  │                 │  │                     │
│ AI/BI Dashboards│  │ Feature Eng.    │  │ Delta Sharing       │
│ Genie Spaces    │  │ Model Training  │  │ Clean Rooms         │
│ Databricks Apps │  │ MLflow Registry │  │ Marketplace         │
│                 │  │ Model Serving   │  │ Lakehouse Federation│
│                 │  │ Online Tables   │  │                     │
│                 │  │ Vector Search   │  │                     │
│                 │  │ Agent Framework │  │                     │
│                 │  │ AI Gateway      │  │                     │
└────────┬────────┘  └─────────────────┘  └─────────────────────┘
         │
         ▼
  Databricks One
  (Business User Portal)
```

**Governance throughout:** Unity Catalog governs every layer — access control (ACLs, ABAC, row filters, column masks), tagging, lineage, audit logging, and data quality monitoring. Tags applied at Bronze propagate through lineage; ABAC policies enforce access dynamically at query time. See the companion **EDAP Tagging Strategy** for the full governed tag taxonomy applied across all layers.

**Data contracts:** The platform's data contract capability is built from the combination of SDP expectations (quality guarantees), UC Metrics (semantic business definitions), governed tags (classification and sensitivity), and system tables (freshness and completeness monitoring). Together, these provide the machine-enforceable interface between data producers and consumers described in the companion **Domain Governance Across Systems** document.

**Orchestration throughout:** Lakeflow Jobs ties the entire pipeline together as a single DAG — ingestion task → Bronze SDP pipeline → Silver SDP pipeline → Gold SDP pipeline → dashboard refresh. Table-update triggers enable event-driven execution across layers.

**Compute throughout:** Serverless compute (secured by Lakeguard) is the default for SDP pipelines, SQL warehouses, notebooks, and jobs — providing seconds-level start-up, automatic scaling, and unified security enforcement.

---

## 11. What's New and What's Coming

| Feature | Status (Mar 2026) |
|---|---|
| Lakeflow Connect (Salesforce, Workday, SQL Server, GA4, ServiceNow, SharePoint, NetSuite) | GA |
| Lakeflow Connect (PostgreSQL, MySQL) | Public Preview |
| Lakeflow Connect (Confluence, SFTP, MongoDB, DB2, DynamoDB) | Preview / Roadmap |
| Spark Declarative Pipelines (SDP) — serverless | GA |
| SDP in open-source Apache Spark 4.1 | GA |
| Unity Catalog ABAC | Public Preview (account-level) |
| Unity Catalog Metrics | Public Preview |
| Lakehouse Federation | GA |
| Iceberg REST Catalog API (read) | GA |
| Iceberg REST Catalog API (write) | Public Preview |
| Iceberg managed tables | Public Preview |
| Predictive Optimization | GA |
| Clean Rooms | GA |
| Marketplace | GA |
| AI/BI Dashboards | GA |
| AI/BI Genie | GA |
| Genie Research Agent / Agent Mode | Beta |
| Genie Code for dashboard authoring | Public Preview |
| Dashboard embedding (external users) | GA |
| Dashboard subscriptions to Teams | Public Preview |
| Data Quality Monitoring (anomaly detection) | GA |
| Lakeguard (shared/serverless compute isolation) | GA |
| AI Gateway | GA |
| Online Tables | GA |
| Delta Sharing for Iceberg clients | Preview |
| Legacy features disabled for new accounts (Dec 2025+) | Enforced |

---

## 12. Companion Documents

This platform capabilities document is part of a broader wiki collection. The following companion documents describe how these platform capabilities are applied within Water Corporation's enterprise context:

| Document | Relationship |
|---|---|
| **EDAP Access Model** | Defines the layer-based catalog structure (`<env>_<layer>`) and access control patterns built on Unity Catalog |
| **Medallion Architecture** | Details the zone decomposition within each medallion layer and the data flow conventions |
| **Enterprise Data Models** | Describes the dimensional and entity models that populate the Gold layer |
| **EDAP Tagging Strategy** | Defines the four-layer governed tag taxonomy applied to all Unity Catalog assets |
| **Domain Governance Across Systems** | Three-layer governance architecture spanning Unity Catalog, Alation, and organisational process — includes data contracts, Delta Sharing governance, and Lakehouse Federation governance |
| **Data Engineering Lifecycle** | Lifecycle framework for building and operating data pipelines on this platform |
| **Data Science Lifecycle** | Lifecycle framework for ML/AI development using Mosaic AI capabilities |
| **BI Lifecycle** | Lifecycle framework for analytics and reporting using AI/BI Dashboards and Genie |
| **Data Governance Lifecycle** | End-to-end governance lifecycle from classification through compliance |

---

## 13. Key References

- [Lakeflow Connect Documentation](https://docs.databricks.com/aws/en/ingestion/lakeflow-connect/)
- [Spark Declarative Pipelines](https://docs.databricks.com/aws/en/ldp/)
- [Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog/)
- [Lakehouse Federation](https://docs.databricks.com/aws/en/query-federation/)
- [AI/BI Dashboards and Genie](https://docs.databricks.com/aws/en/ai-bi/)
- [Mosaic AI](https://docs.databricks.com/aws/en/machine-learning/)
- [AI Gateway](https://docs.databricks.com/aws/en/ai-gateway/)
- [Data Quality Monitoring](https://docs.databricks.com/aws/en/data-quality-monitoring/)
- [Lakeguard](https://docs.databricks.com/aws/en/compute/lakeguard)
- [Lakeflow Jobs](https://docs.databricks.com/aws/en/jobs)
- [Delta Sharing](https://docs.databricks.com/aws/en/delta-sharing/)
- [Clean Rooms](https://docs.databricks.com/aws/en/clean-rooms/)
- [Marketplace](https://docs.databricks.com/aws/en/marketplace/)

---

*Document compiled from Databricks documentation, product blogs, and release notes current as at March 2026. Feature availability may vary by cloud provider and region.*
