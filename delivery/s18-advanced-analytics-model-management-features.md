# S18 – Advanced Analytics Model Management: Feature Breakdown

**Scope Area:** Use Case Delivery
**Delivery Method:** SAFe / Scrum
**Aligned Wiki References:**
- `lifecycles/data-science-lifecycle.md` — MLOps stages, Mosaic AI, feature stores, model serving, monitoring
- `lifecycles/data-governance-lifecycle.md` — AI model governance stage, model inventory, risk classification
- `governance/data-governance-roles.md` — AI/ML Governance Lead, Data Product Owner, model accountability
- `platform/edap-access-model.md` — Unity Catalog model registry, service principals, compute modes

---

## Feature S18-F1: Data Scientists Track Experiments and Compare Model Runs

**Description:** Data scientists can log every experiment — parameters, metrics, and artefacts — to MLflow, compare model runs side by side, and register the best models in Unity Catalog with full version history and lineage, giving the team a governed, reproducible foundation for all model development.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S18-F1-US01 | Data Scientist | log experiments (parameters, metrics, artefacts) to MLflow tracking within the EDAP workspace | I can compare model runs, reproduce results, and share findings with my team |
| S18-F1-US02 | Data Scientist | register trained models in the Unity Catalog model registry with version numbers and stage transitions | models are governed assets with lineage, access control, and lifecycle management |
| S18-F1-US03 | AI/ML Governance Lead | view a model inventory showing all registered models, their versions, training data sources, and current deployment stage | I can maintain the AI model register required by WC's AI governance framework |
| S18-F1-US04 | Data Platform Engineer | configure the MLflow tracking server and Unity Catalog model registry as shared infrastructure accessible from the development and production workspaces | data scientists have a consistent, centrally managed MLOps environment |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S18-F1-AC01 | A data scientist runs a training experiment in the dev workspace | the experiment completes | all parameters, metrics (e.g. RMSE, AUC), and artefacts (e.g. model binary, feature importance plots) are logged to MLflow and visible in the experiment UI |
| S18-F1-AC02 | A trained model is registered in Unity Catalog | the model is queried via `SELECT * FROM system.information_schema.registered_models` | the model appears with its name, version, creation timestamp, and registered by identity |
| S18-F1-AC03 | A model is promoted from version 1 to version 2 in the registry | the model registry is reviewed | both versions are retained, version 2 is marked as the current version, and version 1 remains accessible for rollback |
| S18-F1-AC04 | A model registered in Unity Catalog references training data from `prod_customer.curated` | the model's lineage is inspected | Unity Catalog lineage shows the model's dependency on the source tables used during training |
| S18-F1-AC05 | MLflow tracking and the model registry are configured | a data scientist in the dev workspace and a service principal in the prod workspace both access the registry | both can read the model registry; only the prod service principal can deploy models to production endpoints |

### Technical Notes
- Use Unity Catalog as the model registry (not the legacy workspace-level MLflow Model Registry). This ensures models are governed assets with ABAC, lineage, and audit logging.
- MLflow experiments should be organised by domain and use case (e.g. `/customer/omlix/propensity`).
- Model versions in UC support aliases (e.g. `champion`, `challenger`) which should be used for deployment stage management.
- Service principals should own production model deployments — individual user accounts must not be used for production serving.
- Align to the data science lifecycle wiki's Stage 4 (Model Training and Evaluation) and Stage 5 (Model Registration).

---

## Feature S18-F2: Features Registered, Versioned, and Reusable Across Models

**Description:** Data scientists and engineers can register, version, and reuse engineered features through Unity Catalog feature tables — with point-in-time correctness, scheduled refresh, and the same ABAC governance applied to any other data asset — so that feature work done once benefits every model that needs it.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S18-F2-US01 | Data Scientist | create and register feature tables in Unity Catalog that store engineered features for the OMLIX algorithms | features are versioned, discoverable, and reusable across experiments and models |
| S18-F2-US02 | Data Scientist | use Mosaic AI Feature Engineering to compute features with point-in-time lookups | training data is free from data leakage and features are computed consistently between training and inference |
| S18-F2-US03 | Data Engineer | schedule feature computation pipelines that refresh feature tables on a defined cadence | models always have access to up-to-date features without manual refresh |
| S18-F2-US04 | Data Domain Steward | apply governance tags and access controls to feature tables in the same way as other Unity Catalog tables | feature data containing sensitive information is subject to the same ABAC policies as source data |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S18-F2-AC01 | A feature table for the OMLIX propensity model has been created in `prod_customer.curated` | the table is queried in Unity Catalog | it is registered as a feature table, with a primary key defined and feature metadata (descriptions, tags) populated |
| S18-F2-AC02 | A data scientist creates a training dataset using Mosaic AI Feature Engineering with a point-in-time join | the training dataset is generated | no future data leaks into the training set — the join respects the timestamp column and only includes features available at the label event time |
| S18-F2-AC03 | A feature computation pipeline is scheduled to run daily via Lakeflow Jobs | 24 hours elapse after source data is updated | the feature table reflects the latest source data, with a pipeline run logged and no data quality failures |
| S18-F2-AC04 | A feature table contains columns derived from PI data | the ABAC policies on the parent catalogue are evaluated | PI-derived feature columns are masked for unauthorised users, consistent with the masking applied to the source PI columns |
| S18-F2-AC05 | Three feature tables are registered for the three OMLIX algorithms | the feature tables are inspected in Unity Catalog | each table has a primary key, feature descriptions, lineage to source tables, and appropriate governed tags applied |

### Technical Notes
- Feature tables in Unity Catalog are standard Delta tables with additional metadata (primary key, timestamp key for point-in-time lookups).
- Use `databricks.feature_engineering.FeatureEngineeringClient` for feature table creation and training set generation.
- Feature tables should reside in the `curated` schema of the relevant domain catalogue (e.g. `prod_customer.curated.omlix_features_*`).
- Point-in-time correctness is critical for the OMLIX algorithms — validate by comparing training set results with and without time filtering.
- Align to the data science lifecycle wiki's Stage 3 (Feature Engineering) guidance on feature stores and governed features.

---

## Feature S18-F3: Model Training Reproducible and Automated

**Description:** Data scientists can trigger a training pipeline — on demand or on schedule — that reads from governed feature tables, tunes hyperparameters, logs everything to MLflow, and registers the best model in Unity Catalog, producing a fully reproducible and auditable training run every time.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S18-F3-US01 | Data Scientist | execute a training pipeline that trains an OMLIX model, logs the experiment to MLflow, and registers the resulting model in Unity Catalog | the full training workflow is automated and reproducible |
| S18-F3-US02 | Data Scientist | run hyperparameter tuning (e.g. via Hyperopt or Optuna) as part of the training pipeline | the best model configuration is selected systematically rather than through manual experimentation |
| S18-F3-US03 | Data Platform Engineer | configure GPU-enabled compute clusters for training workloads that require them | deep learning or computationally intensive algorithms can train within acceptable timeframes |
| S18-F3-US04 | Data Scientist | trigger a retraining run on demand or on a schedule when new data is available | models remain current as underlying data patterns evolve |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S18-F3-AC01 | A training pipeline for OMLIX Algorithm 1 is configured as a Lakeflow Job | the job is triggered | it reads features from the feature table, trains the model, logs all parameters and metrics to MLflow, and registers the model in Unity Catalog — all without manual intervention |
| S18-F3-AC02 | Hyperparameter tuning is configured for an OMLIX algorithm | the tuning job completes | at least 20 parameter combinations are evaluated, the best run is identified by the primary metric, and the best model is automatically registered |
| S18-F3-AC03 | A GPU cluster policy is defined for ML training workloads | a data scientist submits a training job requiring GPU compute | the job runs on a GPU-enabled cluster that conforms to the cluster policy (instance type, autoscaling limits, spot instance usage) |
| S18-F3-AC04 | A scheduled retraining job is configured to run monthly | one month elapses | a new model version is trained, evaluated against the current champion model's metrics, and registered in the model registry with a `challenger` alias |
| S18-F3-AC05 | All three OMLIX algorithm training pipelines are deployed | the pipelines are reviewed | each pipeline is parameterised for environment (dev/staging/prod), uses feature tables as input, logs to MLflow, and registers models in Unity Catalog |

### Technical Notes
- Training pipelines should be implemented as multi-task Lakeflow Jobs with stages: feature retrieval → training → evaluation → registration.
- Use Databricks Runtime for Machine Learning (ML Runtime) which includes pre-installed ML libraries.
- Hyperparameter tuning can use MLflow's integration with Hyperopt (built into Databricks) or Optuna.
- GPU cluster policies should specify allowed instance types (e.g. `g5.*` on AWS), enforce autoscaling limits, and prefer spot instances for cost optimisation.
- Model promotion from `challenger` to `champion` should require explicit approval — automated promotion is not appropriate for production models without human review.
- Align to the data science lifecycle wiki's Stage 4 (Model Training and Evaluation) and the governance lifecycle wiki's AI model governance stage.

---

## Feature S18-F4: Models Served as Real-Time Endpoints with Traffic Management

**Description:** Trained models are deployed to production endpoints where they serve predictions via REST API — with A/B traffic routing for safe rollouts, infrastructure-as-code provisioning, and a governance review gate ensuring no model reaches production without documented approval.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S18-F4-US01 | Data Scientist | deploy a registered model to a Mosaic AI Model Serving endpoint | the model is available for real-time or batch inference via a REST API |
| S18-F4-US02 | Data Scientist | configure A/B testing by routing a percentage of inference traffic to a challenger model | I can evaluate a new model version against the champion in production before full rollover |
| S18-F4-US03 | Data Platform Engineer | manage serving endpoint configuration (compute size, scaling, auto-shutdown) via infrastructure as code | endpoint provisioning is repeatable, auditable, and version-controlled |
| S18-F4-US04 | AI/ML Governance Lead | approve model deployment to production through a defined review and sign-off process | no model reaches production without governance review, risk assessment, and documented approval |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S18-F4-AC01 | An OMLIX model version is registered in Unity Catalog with the `champion` alias | the model is deployed to a Mosaic AI Model Serving endpoint | the endpoint is accessible via REST API, returns predictions within the defined latency SLA (e.g. <500ms for real-time, <1 hour for batch), and logs all inference requests |
| S18-F4-AC02 | A challenger model is deployed alongside the champion | traffic routing is configured as 90% champion / 10% challenger | inference requests are routed according to the configured split, and both models' predictions are logged for comparison |
| S18-F4-AC03 | An endpoint is provisioned via Databricks Asset Bundles (DABs) or Terraform | the infrastructure-as-code definition is applied | the endpoint is created with the specified compute size, scaling policy, and model version — matching the IaC definition exactly |
| S18-F4-AC04 | A model deployment request is submitted for production | the AI/ML Governance Lead reviews the request | the deployment proceeds only after documented approval that includes: model risk tier, training data lineage, evaluation metrics, bias assessment (where applicable), and rollback plan |
| S18-F4-AC05 | A serving endpoint is running in production | the endpoint is monitored | auto-scaling responds to traffic within 60 seconds, and the endpoint auto-shuts down during zero-traffic windows if configured to do so |

### Technical Notes
- Mosaic AI Model Serving provides serverless, GPU-enabled endpoints with built-in autoscaling.
- Use Unity Catalog model aliases (`champion`, `challenger`) to control which version is served — endpoint configuration references the alias, not a specific version number.
- A/B testing is configured via traffic routing in the serving endpoint configuration. Log both models' predictions to an inference table for offline comparison.
- Endpoint provisioning should be managed via DABs (Databricks Asset Bundles) to ensure consistency across environments.
- The governance review process should align to the AI model governance stage in the governance lifecycle wiki, including risk classification (low/medium/high/critical).
- Service principals should own serving endpoints — not individual user accounts.

---

## Feature S18-F5: Model Performance Monitored with Drift Alerts

**Description:** Data scientists and governance leads can see at a glance whether deployed models are healthy — with automated drift detection, performance tracking against ground truth, and alerts that fire before degradation affects business outcomes.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S18-F5-US01 | Data Scientist | monitor prediction drift (feature drift and concept drift) for deployed OMLIX models | I am alerted when model inputs or outputs shift significantly from the training distribution |
| S18-F5-US02 | Data Scientist | track model performance metrics (e.g. accuracy, precision, recall) over time against labelled ground truth when available | I can detect model degradation and trigger retraining before business impact occurs |
| S18-F5-US03 | AI/ML Governance Lead | view a monitoring dashboard for all deployed models showing health status, drift alerts, and performance trends | I can report on model health to the Data Governance Council and identify models requiring attention |
| S18-F5-US04 | Data Platform Engineer | configure Lakehouse Monitoring on inference tables to compute statistical profiles and drift metrics automatically | monitoring is automated and does not require custom code for each model |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S18-F5-AC01 | An OMLIX model is deployed and logging predictions to an inference table in Unity Catalog | Lakehouse Monitoring is enabled on the inference table | profile metrics (mean, stddev, distribution) and drift metrics (PSI, KS test) are computed on a scheduled basis and stored in monitoring tables |
| S18-F5-AC02 | Feature drift exceeds a configured threshold (e.g. PSI > 0.2 for a key feature) | the monitoring pipeline runs | an alert is generated and sent to the model owner and data science team within 1 hour of detection |
| S18-F5-AC03 | Ground truth labels become available for a batch of predictions | the labels are joined to the inference table | model performance metrics (accuracy, precision, recall, F1 or equivalent) are computed and logged, with a comparison to the baseline performance at deployment time |
| S18-F5-AC04 | A model monitoring dashboard has been deployed | the AI/ML Governance Lead views the dashboard | it displays: model name, version, deployment date, current drift status (green/amber/red), performance trend, last monitoring run timestamp, and alert history |
| S18-F5-AC05 | All three OMLIX models are deployed and monitored | 30 days of monitoring data is accumulated | monitoring history shows continuous drift and performance tracking with no gaps in scheduled monitoring runs |

### Technical Notes
- Lakehouse Monitoring (part of Unity Catalog) can be enabled on any Delta table, including inference tables. It computes statistical profiles and drift metrics automatically.
- Inference tables should be stored in the domain catalogue (e.g. `prod_customer.product.omlix_algo1_inference`) and governed by the same ABAC policies as other domain tables.
- Drift detection should use Population Stability Index (PSI) for feature distributions and Kolmogorov-Smirnov (KS) tests for continuous features.
- Performance monitoring requires ground truth labels — define the process for obtaining and joining labels (may be delayed for some use cases).
- Alerting should use Databricks SQL alerts or integration with WC's monitoring tooling.
- Align to the data science lifecycle wiki's Stage 7 (Model Monitoring) and the governance lifecycle wiki's AI model governance stage.

---

## Feature S18-F6: Three OMLIX Algorithms Operational in the Customer Domain

**Description:** The three OMLIX algorithms are trained, deployed, and producing inference results that are integrated into the Customer Data Mart — so that data consumers can query algorithm predictions and scores from Gold-layer tables and use them in reports and operational processes.

### User Stories

| Story ID | As a... | I want to... | So that... |
|---|---|---|---|
| S18-F6-US01 | Data Product Owner (Customer Domain) | have the three OMLIX algorithms trained on Customer Domain data and producing inference results that are integrated into the Customer Data Mart | the Customer Data Mart data product includes advanced analytics outputs for downstream consumers |
| S18-F6-US02 | Data Scientist | develop, train, and validate each OMLIX algorithm using the feature engineering and MLOps infrastructure established in S18-F1 through S18-F5 | algorithm development follows the governed MLOps process rather than ad-hoc notebook execution |
| S18-F6-US03 | Data Consumer | query OMLIX inference results from the Customer Data Mart Gold-layer tables | I can use algorithm predictions and scores in reports and operational processes |
| S18-F6-US04 | AI/ML Governance Lead | review each OMLIX algorithm's risk classification, training data lineage, and performance metrics in the model inventory | each algorithm is documented and governed per the AI governance framework |

### Acceptance Criteria

| AC ID | Given | When | Then |
|---|---|---|---|
| S18-F6-AC01 | OMLIX Algorithm 1 has been developed, trained, and registered | inference is executed on the Customer Domain data | inference results are written to a Gold-layer table in `prod_customer.product` with an inference timestamp and model version identifier |
| S18-F6-AC02 | OMLIX Algorithm 2 has been developed, trained, and registered | inference is executed | results meet the agreed-upon accuracy threshold defined in the algorithm specification and are available in the Customer Data Mart |
| S18-F6-AC03 | OMLIX Algorithm 3 has been developed, trained, and registered | inference is executed | results are produced, written to the Gold layer, and the inference pipeline completes within the defined SLA (e.g. daily batch within a 4-hour processing window) |
| S18-F6-AC04 | All three OMLIX algorithms are deployed | the model inventory is reviewed | each algorithm has a documented entry including: model name, version, risk tier, training data sources, feature tables used, evaluation metrics, deployment date, and responsible domain owner |
| S18-F6-AC05 | OMLIX inference results are available in the Customer Data Mart | a data consumer queries the Gold-layer table | the results include: entity key (e.g. customer ID), prediction/score, confidence interval (where applicable), inference timestamp, and model version |

### Technical Notes
- Each OMLIX algorithm should be implemented as a separate model in the Unity Catalog model registry, with its own training pipeline (S18-F3), serving configuration (S18-F4), and monitoring setup (S18-F5).
- Inference may be batch (scheduled Lakeflow Job) or real-time (Mosaic AI Model Serving endpoint) depending on the algorithm requirements — confirm with the Customer Domain Data Product Owner.
- Inference output tables in `prod_customer.product` must include audit columns (`edap_ingested_at`, `edap_model_version`) per the medallion architecture standards.
- Apply governed tags to inference tables (e.g. `sensitivity`, `pi_category` if outputs could be used to infer PI).
- Align algorithm risk classification to the governance lifecycle wiki's AI model risk tiers: low, medium, high, critical.
