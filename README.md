# Ignite 2026: supply-chain ontology workshop

Set up a Microsoft Fabric workshop from one notebook and publicly hosted files.
All business data is synthetic. Participants start with a semantic model, then
discover additional entities and operational observations through an ontology.

For the full walkthrough, use the [end-to-end guide](TRY-IT-END-TO-END.md)
and [discovery-first prompt card](prompts/workshop.md).

## Optional: use your own local agent

Your local coding or chat agent can guide the workshop one step at a time.
Open a local copy of this repository in your agent, or provide the
[agent guide](AGENT-GUIDE.md) and its linked README, walkthrough and prompt card.
Then send:

> Read AGENT-GUIDE.md and use it to guide me through this workshop one step at a
> time. First establish where I have reached and which workspace and folder I
> intend to use. Wait for my results and approval before changes.

No particular agent product or Fabric integration is required. Without Fabric
tools, your agent explains the UI steps and prompts while you perform them.
With authorized integrations, it can help inspect and operate your environment
within the guide's approval gates. It must not ask you to paste credentials or
claim checks passed without evidence. Start a normal guided conversation; no
special agent installation or automatic deployment is needed.

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

> Where is our supply chain most vulnerable?

Follow an example with "Why is that a risk? Show me what depends on it." If
the current ontology cannot answer, ask "What additional data would help us
answer this?" Use that gap to introduce Lakehouse enrichment:

> Find the supply-chain dependencies in our Zava lakehouse that aren't in this
> ontology yet. Add the new business objects and connect them to what we already
> have. Keep the existing model and measures. Show the plan first.

Review the six new entities and the 11 connections in
[the dependency contract](data/dependencies/contract.json). Reuse existing
Product, Component, Supplier, Facility and Carrier entities; Facility's key is
ObjectId. Only switch to **Act** to apply the reviewed plan, then return to Plan.

Return to the opening question after enrichment:

> Where is our supply chain most vulnerable?

Follow what the answer reveals, one question at a time:

> Why is that a risk? Show me what depends on it.

> Do we have a genuine alternative?

> Which customer orders could be affected?

Discover snapshot times from the data and state timing assumptions rather than
requiring participants to supply dates, identifiers or a recovery horizon.
Trace both primary-component links and additional bills of material. Check
qualification, lead time, capacity and shared dependencies before calling an
alternative usable. These are upstream
replenishment dependencies, not proof of the cause of existing shipment delays,
lost sales or an executable recovery plan. An independent option may lack
qualification or exceed the recovery horizon. LeadTimeHours already includes
TransitHours; do not add transit again. Deduplicate orders across multiple paths.

For the Eventhouse exercise, preview shipment delay/shock/route-risk observations
and inventory availability/days-of-supply/stockout-risk observations. Join to
existing ShipmentId and InventoryPositionId using ObservationTime. Keep the
existing semantic grounding and review the plan before applying. In Plan, ask:

> Which deliveries need our attention?

Follow the cases identified in the answer:

> What makes these stand out?

> Should we respond to all of them the same way?

> What have we already decided, and what still needs checking?

Discover the available observation times, select a bounded window within the
data and use the latest observations per identity within it. Report the chosen
times and freshness differences; do not use today's clock. Missing observations
are unknown, not zero risk. Distinguish recorded Approved/Proposed decisions from
actions actually executed.

For either exploration, ask:

> Show me the data behind that.

Inspect actual source queries and separate facts from interpretation. Follow
the conversation rather than forcing predetermined examples or identifiers.

## Files and maintenance

| Path | Purpose |
|---|---|
| `Stage_Zava_Workshop.ipynb` | Reviewed deployment code, HTTP loader and parameters; no embedded data |
| `manifest.json` | Allowed source paths, lengths, checksums and base deployment identity |
| `TRY-IT-END-TO-END.md` | Setup, source checks, ontology enrichment and discovery-first walkthrough |
| `prompts/workshop.md` | Short business questions, follow-ups and reviewed Plan/Act prompts |
| `AGENT-GUIDE.md` | Optional provider-neutral instructions for a participant's own local agent |
| `data/tables` | Original 13 business-table files |
| `data/dependencies` | Six additional entities, schemas and relationship mappings |
| `data/telemetry` | Four time-series fixtures |
| `data/contract.json`, `data/schemas.json`, `data/telemetry-schemas.json` | Base table contract and schemas |
| `model` | Portable semantic-model definition |

Publish data changes and the matching regenerated checksum manifest together.
Compatible data-only updates do not require rebuilding the notebook. Source
revision and manifest identity are recorded in the private deployment ledger.
Fixture bytes are preserved by `.gitattributes`, keeping byte-based ownership
compatible with the earlier embedded-data notebook. Dataset identity is computed
from file contents, independently of Python/ZIP compression versions; only the
exact original dataset retains its legacy ownership identity.

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
ontology/agent journey. This public repository contains the installer, relevant
data/model files and participant guides, not local tests or private live evidence.
