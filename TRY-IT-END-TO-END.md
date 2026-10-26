# Try the Zava workshop end to end

**Goal:** start with an empty Fabric workspace, stage the sources, create your
own ontology and explore supply-chain vulnerabilities and delivery responses.

**Before you begin:** fresh staging into an empty deployment folder has completed
in Fabric, with all 19 SQL tables checked and the unchanged 13-table model
returning its expected totals. A completely empty-workspace deployment and the
full ontology UI journey remain separate rehearsal gates.

## Optional: let your own local agent guide you

You can follow this guide yourself or have a local coding/chat agent walk you
through it. Give the agent a local copy of this repository, or provide
[AGENT-GUIDE.md](AGENT-GUIDE.md) and its linked companion documents, then send:

> Read AGENT-GUIDE.md and guide me through the workshop one step at a time.
> Establish my current progress and intended workspace and folder first.
> Explain what to do, wait for my results, and ask before making changes.

Fabric integrations are optional: without them, you perform the UI actions and
share relevant outputs or redacted screenshots. If integrations are available,
the agent must confirm scope and obtain approval before authoring or running
setup. Never paste tokens or credentials. You still review every ontology plan
before applying it; the guide does not authorize unattended deployment.

## 1. Get the starter files

Open [Stage_Zava_Workshop.ipynb](Stage_Zava_Workshop.ipynb) in this repository
and select **Download raw file**. The notebook is the only file you upload to
Fabric. It downloads the data files and semantic-model definition from the
public repository over HTTPS, verifying their lengths and SHA-256 checksums.
No Git client, GitHub credentials or ZIP extraction is needed.

The Spark driver needs access to `raw.githubusercontent.com` and, when resolving
a branch or tag, `api.github.com`. No remote Python is downloaded or executed.
Do not copy source IDs or run another demo's bootstrap.

Use the [prompt card](prompts/workshop.md) alongside this guide for the
conversation steps.

## 2. Create a new, empty workspace

Create a disposable Fabric workspace with a name you can easily recognize.
Assign it to an active Fabric capacity or supported trial.

You need Contributor or higher and the relevant Fabric/ontology preview features
enabled. Use your normal signed-in user account.

**Recommended:** use a dedicated disposable workspace for this first rehearsal.
Folder mode can also create independent sources in an existing workspace.

Choose either:

- **Workspace root:** leave it empty; setup will create **SupplyChain**.
- **Folder mode:** create a folder, such as **Ignite2026**, and put the imported
  notebook there before its first run. Setup deploys directly beside it, not in
  another SupplyChain subfolder. Source names get a notebook-specific suffix,
  so an older demo elsewhere is not reused. Unrelated items remain unchanged.

Below, "deployment folder" means your chosen folder, or SupplyChain in root mode.

## 3. Import the notebook and preview

Use the workspace's **Import / Upload notebook** option to import
`Stage_Zava_Workshop.ipynb`, then open it as a **PySpark** notebook.
For folder mode, move the imported notebook into your chosen folder first.

Do **not** attach a default Lakehouse.

Leave the parameters as:

```python
APPLY = False
CONFIRM_WORKSPACE_NAME = ""
REPO_REVISION = "main"
```

Select **Run all**.

**Expect:** the notebook prints your workspace name, target folder and a deployment preview.
Check that the target folder is the one you intended. The output also records
the resolved Git commit; all input files come from that same immutable revision.
It should not create sources or ingest data. Spark startup can still use capacity.

## 4. Run setup

In the parameters cell, change:

```python
APPLY = True
CONFIRM_WORKSPACE_NAME = "Your exact workspace name from the preview"
```

For a repeatable run, also replace `REPO_REVISION` with the full commit printed
by preview. Leaving it as `"main"` resolves the branch again; its contents may
have changed between preview and apply.

Select **Run all** again. Keep the tab open and allow provisioning and refresh
to finish. Do not start a second run while this one is working.

**Expect:** these sources in your deployment folder:

| Item | What it provides |
|---|---|
| ZavaSupplyChain2027_Lakehouse_Source | 19 managed Delta tables, including six dependency tables absent from the model |
| ZavaSupplyChain2027_Eventhouse_Source | Source Eventhouse and KQL database with four compact telemetry tables |
| ZavaSupplyChain2027_SemanticModel | Starting model with 13 business tables and 12 measures |

The final notebook message should report sources staged and model refresh
completed, with a DAX smoke check still required. An exception is a stop condition,
not a partially successful setup to work around.

The table shows base names. In folder mode, use the full **printed source names**
ending in `_Lab_...` for the remaining steps. For example, generate your ontology
from the newly staged semantic model with that suffix, not an older unsuffixed model.

## 5. Check that the data and model work

Open the Lakehouse's **SQL analytics endpoint**. Wait for the 19 lowercase
`dbo` tables to appear. Run:

```sql
SELECT TOP (2) ShipmentId, CustomerOrderId, ShipmentQuantity
FROM dbo.shipment
WHERE ShipmentId IN ('SHIPMENT-0001', 'SHIPMENT-0002')
ORDER BY ShipmentId;
```

**Expect:** both shipments, each with quantity **120**. SQL metadata can lag;
wait for synchronization rather than rerunning ingestion.

Next, open the semantic model's **...** menu or details page and select
**Write DAX queries**. Paste and run:

```dax
EVALUATE
ROW(
    "Products", [Product Count],
    "OnHandUnits", [On Hand Inventory Units],
    "NetAvailableUnits", [Net Available Inventory Units],
    "AtRiskOrderUnits", [At Risk Customer Order Units]
)
```

**Expect:** `80`, `100700`, `72100`, `3534`.

If asked to configure source credentials, use your organizational identity and
the Lakehouse created in this workspace. Do not select the original demo's source.
Do not continue until the model can return data.

## 6. Create your ontology from the model

On **ZavaSupplyChain2027_SemanticModel**, select **Generate Ontology**.
It may be under **...**.

Name it:

```text
Ignite2026_Zava_MyTest
```

Create it in your **deployment folder**. If generation places it at the workspace root,
move only your new ontology into that folder before continuing.

**Expect:** concepts corresponding to the model's 13 business tables. Inspect
Shipment, InventoryPosition, CustomerOrder, Disruption, Intervention and
DecisionRecord. Review keys, relationships and metrics; do not activate all
inactive relationships.

No reference ontology is installed by the starter. Creating this ontology is
part of the exercise.

## 7. Ask a first business question

Open the **Ontology agent** and select **Plan** mode.

Set the context:

> We are working on my Zava workshop ontology in my deployment folder. Keep the source
> data and semantic model unchanged. Discover the available snapshot and observation
> times from the data rather than using today's clock. Show the dates, analysis
> window and assumptions behind time-sensitive answers.

Then ask:

> Where is our supply chain most vulnerable?

Follow an example from the answer:

> Why is that a risk? Show me what depends on it.

If the current grounding cannot answer:

> What additional data would help us answer this?

**Expect:** a model-backed answer with a supporting query. This is the starting
static view, not yet evidence that telemetry was used.

## 8. Discover Lakehouse-only entities and dependencies

Stay in **Plan** and ask:

> Find the supply-chain dependencies in our Zava lakehouse that aren't in this
> ontology yet. Add the new business objects and connect them to what we already
> have. Keep the existing model and measures. Show the plan first.

Review the preview. It should add **BillOfMaterial**, **SupplierSite**,
**ComponentSupply**, **SupplyRoute**, **RouteHub** and **LogisticsHub** using the
Lakehouse, not by changing the semantic model. Check their own identifiers and
11 relationships against the
[dependency contract](https://github.com/adamlash/ontology-ignite-2026/blob/main/data/dependencies/contract.json).

Existing Product, Component, Supplier, Facility and Carrier must be reused.
Facility joins use **ObjectId**. Keep original Shipment/InventoryPosition backing
and all semantic metrics; do not duplicate existing objects or keys.

If the additive plan looks correct, switch to **Act** and say:

> Apply only the plan we just reviewed. Keep everything else unchanged.

**Switch back to Plan afterward.**

Return to the opening question with the enriched model:

> Where is our supply chain most vulnerable?

Follow what the answer reveals, one question at a time:

> Why is that a risk? Show me what depends on it.

> Do we have a genuine alternative?

> Which customer orders could be affected?

**Expect:** the answer follows multiple hops through new and existing entities,
checks qualification and lead time, and deduplicates orders. This is dependency
exposure, not predicted lost sales or proof of the cause of the earlier delays.
Check capacity and shared dependencies as well, and make timing assumptions explicit.
The traversal should include primary-component links and bills of material.
Inspect its actual source queries. A plausible diagram alone is not a pass.

## 9. Add the Eventhouse observations

In **Plan**, ask:

> Add shipment delay, handling shock and route risk, plus inventory availability,
> days of supply and stockout risk, from the Zava source Eventhouse. Match the
> existing objects and use observation time. Show the plan first.

Before applying, check:

| Existing concept  | Source table               | Matching key        | Time column     |
| ----------------- | -------------------------- | ------------------- | --------------- |
| Shipment          | ShipmentTelemetry          | ShipmentId          | ObservationTime |
| InventoryPosition | InventoryPositionTelemetry | InventoryPositionId | ObservationTime |

The plan should add six observation properties without replacing the existing
concepts or losing their semantic-model grounding.

If correct, switch to **Act**, apply only the reviewed plan, then return to
**Plan**. Verify that the properties and bindings persisted.

## 10. Discover deliveries and compare responses

In **Plan**, ask:

> Which deliveries need our attention?

Follow the cases it identifies, without supplying shipment IDs:

> What makes these stand out?

> Should we respond to all of them the same way?

> What have we already decided, and what still needs checking?

The agent should discover the available time range, select a bounded window within
the data and show the latest observations per identity within it. Do not use today's
clock or assume that observations from different sources are equally fresh.
These prompts are a conversation, not a requirement to select two predetermined
shipments. If an answer needs substantiation, ask:

> Show me the data behind that.

Then ask:

> Show the actual queries, sources and observation times behind that answer.
> Separate recorded facts from your interpretation.

**Optional facilitator example; do not paste it into the agent.** If discovery
selects these shipments, use the table below. Other cases need their own verified
source evidence; they are not failures merely because their identifiers differ.

| Evidence | SHIPMENT-0001 | SHIPMENT-0002 |
|---|---|---|
| Units involved | 120 | 120 |
| Available-to-promise inventory | 180 | 0 |
| Observed shock | 3g | 12g |
| Estimated delay | 45 hours | 45 hours |
| Recorded intervention | Reroute | HoldAndInspect |
| Recorded decision state | Approved | Proposed |
| Shipment observation UTC | 18:01 | 18:02 |
| Inventory observation UTC | 17:41 | 17:42 |

**Expect:** the answer distinguishes stock-supported reroute discussion from a
proposed quality hold. It must not claim either action was executed or that
on-time delivery was proved. The shock values are observations, not documented
policy thresholds.

This example's evidence window is **17:00-20:00 UTC on 31 December 2027**, not
today's date. Participants discover and report the relevant dates from the data.
A correct-looking answer without actual source queries is not a full pass.

## If something stops working

| Problem | What to do |
|---|---|
| Setup says the workspace is not blank | Move the notebook into your chosen deployment folder before first setup, or use a new empty workspace; do not delete unrelated items |
| Setup reports a source-name conflict | The exact notebook-specific name already exists without matching ownership; stop and inspect it rather than bypass the guard |
| Setup reports that the notebook moved | Restore its original location; reruns do not relocate sources |
| Download or checksum verification fails | Check network access and the selected revision; do not bypass verification or clear the ownership ledger |
| An older notebook fails on a workspace-wide `/kqlDatabases` read | Preserve its identity, location and ledger; follow the reviewed runtime-update guidance in the repository README rather than importing a second notebook over existing sources |
| Setup fails or reports an uncertain prior write | Stop. Keep the error and `Files/ignite2026/bootstrap-state.json`; do not clear the ledger or repeatedly submit setup |
| Lakehouse looks empty | Wait for SQL synchronization, check permissions and lowercase table names |
| Generate Ontology / agent is unavailable | Check tenant feature settings, capacity and permissions |
| Preview creates duplicate concepts or `_2` keys | Reject it and ask to reuse the existing concepts and properties |
| Telemetry is empty | Check the source database and ObservationTime; discover the available dates before selecting a bounded window instead of filtering relative to today's clock |

For any blocker, record the step number, exact error or agent response, and a
screenshot. Do not include tokens or credentials.

After setup, keep the notebook in its original location so its ownership ledger
can be used safely on reruns.

**Finished means:** setup completed, the model query returned the expected
values, your ontology was generated, reviewed enrichments persisted and the
final answer included verifiable source queries. Graph and sharing are separate
follow-on checks, not required for this first personal rehearsal.
