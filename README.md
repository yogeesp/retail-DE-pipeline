# Azure Sales Order Validation Pipeline
## Overview
End-to-end data pipeline that ingests order data (~70K records) from a third-party drop, validates it, and processes it through Bronze/Silver/Gold layers using Azure Data Factory, Databricks, and Delta Lake.
### Architecture
Third-party file (orders.csv) → ADLS Gen2 (landing) → ADF-orchestrated Databricks notebook → validation → Bronze → Silver → Gold → staging (or discarded on failure)
### Tech Stack
Azure Data Factory, Azure Data Lake Storage Gen2, Databricks (PySpark, Delta Lake), Unity Catalog (Volumes, External Locations, Storage Credentials), Azure SQL Database (JDBC), Azure Key Vault, Databricks Secret Scopes.
### What it does
<ul>
 <li>Ingests ~70K order records dropped as CSV in a landing folder. </li>
<li>Parameterized notebook (Databricks widgets) — handles any filename/catalog/schema/volume without code changes.</li>
<li>Validation layer: checks duplicate order_id, validates order_status against a lookup table in Azure SQL DB via JDBC connection.</li>
<li>Failed records get moved to a discarded folder with structured error messages (JSON); passed records move to staging.</li>
<li>Bronze layer: raw ingested data as Delta table.</li>
<li>Silver layer: deduplicated, status-validated, cleaned data.</li>
<li>Gold layer: aggregated customer-level order counts for downstream analytics.</li>
<li>Secrets (SQL password, storage keys) secured via Azure Key Vault, accessed through Databricks Secret Scopes — never hardcoded.</li>
<li>Unity Catalog governs storage access via Volumes and External Locations.</li>
</ul>

### Known limitations
<ul>
<li>Storage Event Trigger in ADF set up but unreliable — Event Grid doesn't reliably fire on dbutils.fs.mv (rename-style) operations in ADLS Gen2 HNS accounts. Manual trigger used for demo; production fix would test against true blob-create events (e.g. direct upload, not internal move).</li>
<li>Gold layer currently covers order-count metrics only. Spend-per-customer requires joining order_items data (not yet integrated).</li>
</ul>

Sample Results
Processed ~70,000 order records end-to-end; validation logic correctly flagged duplicate order_ids and invalid order_status values, routing them to discarded folder.
