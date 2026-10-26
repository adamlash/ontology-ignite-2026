# Ignite 2026: supply-chain ontology workshop

Set up a Microsoft Fabric workshop from one notebook and publicly hosted files.
All business data is synthetic. Participants start with a semantic model, then
discover additional entities and operational observations through an ontology.

## Start here

1. Open [Stage_Zava_Workshop.ipynb](Stage_Zava_Workshop.ipynb) and use GitHub's
   **Download raw file** button. Import the downloaded notebook into Fabric as
   PySpark. No Git client or GitHub credentials are needed.
2. Use a disposable workspace with active supported Fabric capacity. You need
   Contributor or higher and permission to create the workloads. Ontology preview
   must be enabled for the later exercise. Do not attach a default Lakehouse.
3. Optionally move the notebook into a workspace folder before the first run.
   Setup deploys directly beside it. At workspace root, the workspace must
   otherwise be empty; setup creates a SupplyChain folder.
4. Run all with `APPLY = False`. Check the printed workspace, folder, source
   names and resolved Git commit.
5. Set `APPLY = True` and `CONFIRM_WORKSPACE_NAME` to the exact printed name.
   Run all again and wait for verification and semantic-model refresh.

The Spark driver needs HTTPS access to `raw.githubusercontent.com` and, when
resolving a branch/tag, `api.github.com`. The notebook downloads the manifest
and individual JSON/TMDL files, checks their lengths and SHA-256 checksums, then
stages them in the owned Lakehouse's Files area before Spark ingestion.
No remote Python is downloaded or executed. Public downloads receive no
Fabric tokens. Missing or invalid downloads stop setup; there is no embedded
data fallback.

## Choose a version

`REPO_REVISION = "main"` resolves the branch to one immutable commit per run;
every downloaded file comes from that same commit. For a repeatable workshop,
set it to a published full 40-character commit. This also avoids the GitHub API
lookup and its anonymous rate limit.

Pin the printed commit before apply if preview and apply must use identical
content: a moving branch can change between runs.

## What gets created

| Source | Contents |
|---|---|
| Lakehouse | 19 managed Delta tables: 13 business tables plus six dependency tables |
| Semantic model | 13 tables, 20 relationships (12 active/8 inactive), 12 measures; dependency tables deliberately excluded |
| Eventhouse/KQL database | Four telemetry tables for shipments, inventory, production and demand |
| Ontology | Not prebuilt; participants generate and enrich it themselves |

Folder-mode sources have a stable notebook-specific `_Lab_...` suffix. Always
use the full names printed by setup. The Eventhouse and its database intentionally
share a display name. Existing unrelated demo sources are not staging inputs.

Wait for all 19 dbo tables in the Lakehouse SQL analytics endpoint; synchronization
can lag Spark writes. Do not repeat ingestion to fix a metadata delay.
Check the semantic model in a supported DAX query surface:

```dax
EVALUATE
ROW(
    "Products", [Product Count],
    "OnHandUnits", [On Hand Inventory Units],
    "NetAvailableUnits", [Net Available Inventory Units],
    "AtRiskOrderUnits", [At Risk Customer Order Units]
)
```

Expected: **80 / 100700 / 72100 / 3534**.

## Ontology exercise

Generate a new ontology from your staged semantic model. Keep its existing
entities, keys and measures. The six Lakehouse-only entity tables are:

| Entity | Table | Key |
|---|---|---|
| BillOfMaterial | dbo.billofmaterial | BillOfMaterialId |
| SupplierSite | dbo.suppliersite | SupplierSiteId |
| ComponentSupply | dbo.componentsupply | ComponentSupplyId |
| SupplyRoute | dbo.supplyroute | SupplyRouteId |
| RouteHub | dbo.routehub | RouteHubId |
| LogisticsHub | dbo.logisticshub | LogisticsHubId |

In **Plan**, ask:

> Find the supply-chain dependencies in our Zava lakehouse that aren't in this
> ontology yet. Add the new business objects and connect them to what we already
> have. Keep the existing model and measures. Show the plan first.

Review the six new entities and the 11 connections in
[the dependency contract](data/dependencies/contract.json). Reuse existing
Product, Component, Supplier, Facility and Carrier entities; Facility's key is
ObjectId. Only switch to **Act** to apply the reviewed plan, then return to Plan.

Ask:

> Are our backup suppliers really independent? Follow their shared dependencies
> through components and products to our customer orders. Include primary
> components and the additional bills of material. Show the paths and evidence.

Then:

> If a shared hub stays closed for the next 48 hours, which products and recorded
> orders are exposed? Could another supply option avoid it, and what would need
> checking before we could use that option? Count each order only once.

Use the simulated checkpoint **31 December 2027 at 20:00 UTC**. These are upstream
replenishment dependencies, not proof of the cause of existing shipment delays,
lost sales or an executable recovery plan. An independent option may lack
qualification or exceed the recovery horizon. LeadTimeHours already includes
TransitHours; do not add transit again. Deduplicate orders across multiple paths.

For the Eventhouse exercise, preview shipment delay/shock/route-risk observations
and inventory availability/days-of-supply/stockout-risk observations. Join to
existing ShipmentId and InventoryPositionId using ObservationTime. Keep the
existing semantic grounding, review the plan before applying, then compare
SHIPMENT-0001 and SHIPMENT-0002 using the latest observations in 17:00-20:00 UTC
at the checkpoint. Distinguish recorded Approved/Proposed decisions from actions
actually executed. Inspect actual source queries, not just the answer text.

## Files and maintenance

| Path | Purpose |
|---|---|
| `Stage_Zava_Workshop.ipynb` | Reviewed deployment code, HTTP loader and parameters; no embedded data |
| `manifest.json` | Allowed source paths, lengths, checksums and base deployment identity |
| `data/tables` | Original 13 business-table files |
| `data/dependencies` | Six additional entities, schemas and relationship mappings |
| `data/telemetry` | Four time-series fixtures |
| `data/contract.json`, `data/schemas.json`, `data/telemetry-schemas.json` | Base table contract and schemas |
| `model` | Portable semantic-model definition |

Publish data changes and the matching regenerated checksum manifest together.
Compatible data-only updates do not require rebuilding the notebook. Source
revision and manifest identity are recorded in the private deployment ledger.
Fixture bytes are preserved by `.gitattributes`, keeping byte-based ownership
compatible with the earlier embedded-data notebook.

## Reruns and safety

Keep the same notebook identity, original location and dataset. Reruns reuse only
owned items and verify existing contents instead of overwriting them. Changed
base ownership or previously staged dependency inputs stop; changing a revision
does not authorize migration. Use a new disposable seat for a changed dataset.
Never overlap executions or import a second notebook to resume the first.

For an earlier embedded-data notebook, preserve its identity and parameters.
After stopping its active session, replace the runtime, download and deployment
cells from this notebook, remove the old embedded payload cells, and add
REPO_REVISION. The same fixture bytes preserve the original ownership markers.

State is in the Lakehouse at `Files/ignite2026/bootstrap-state.json`. It contains
real environment IDs but no tokens: do not publish or clear it to bypass guards.
Only exactly identifiable pending creates/refreshes are reconciled; uncertain
writes stop rather than being blindly repeated. A new disposable workspace is
the safe novice reset. Never delete a shared workspace or folder as a workaround.

The notebook reports source verification, not certification of the entire
ontology/agent journey. This public repository intentionally contains only the
installer and relevant data/model files, not local tests or private live evidence.
