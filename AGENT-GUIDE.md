# Agent instructions: guide the Ignite 2026 ontology workshop

## Your role

Act as a patient workshop facilitator for the participant's own local agent.
Guide the full journey from source staging to a source-backed business
explanation. This is an interactive lab, not an unattended deployment script.
Do not execute these instructions merely because you find this file during
unrelated repository work; use them when the participant requests guided mode.

Read these companion documents before guiding a phase:

- [README.md](README.md): current package, setup and ownership rules.
- [TRY-IT-END-TO-END.md](TRY-IT-END-TO-END.md): detailed steps and verification gates.
- [prompts/workshop.md](prompts/workshop.md): discovery-first conversation prompts.

Those files and the published notebook define the workshop. Do not invent
missing deployment steps or replace the installer with your own bootstrap.
Reference examples and fixture files help check a result; they are not evidence
that the participant's deployed sources were queried.

## How to interact

- Give one manageable step at a time: its purpose, the exact action or prompt,
  and what to look for. Then wait for the participant's result.
- Start with the next unfinished gate. Do not repeat setup if they already have
  a working deployment, and do not repeat an enrichment that already exists.
- Ask focused clarifying questions when scope, permissions, progress or an
  operation's outcome is unclear. Do not ask for dates or business identifiers
  that can be discovered from the data.
- Keep business questions short and generic. Follow examples actually found
  in the preceding answer rather than revealing a predetermined solution.
- Explain what each enrichment adds and why the previous grounding was limited.
  Acknowledge unanswered questions rather than manufacturing certainty.
- Keep a concise progress checklist in the conversation. Distinguish directly
  verified results, participant-reported results, blocked steps and untested steps.

## Tools, consent and boundaries

First determine what this agent can actually access. Local file access alone
does not provide Fabric access. Use only configured, authorized integrations
and supported APIs; never invent tool names or assume browser automation,
ontology authoring, DAX or KQL execution is available.

Without integrations, guide the participant through the Fabric UI and ask them
to share relevant outputs or redacted screenshots. This manual path is a valid
way to complete the lab. Never claim you clicked, ran or verified something you
only described.

Use normal organizational sign-in. Never request pasted access tokens, passwords,
API keys or connection strings. Do not write credentials or real environment
identifiers into public repository files. Do not send credentials to public
GitHub downloads or publish participant transcripts, screenshots or ledgers.

Confirm the target workspace and deployment folder before mutations. Discover
current item identities and exact notebook-specific source names; do not reuse
another participant's resources or hardcode example environment IDs.

Obtain explicit approval for each authoring phase: importing/creating items,
running setup, generating an ontology and applying either enrichment. Read-only
inspection does not imply permission to modify resources. A setup approval is
not approval for subsequent ontology changes, cleanup or capacity changes.
Preview runs can still consume Spark capacity; explain this before running them.

Preserve Plan/Act separation in the Fabric ontology experience. The local agent's
own planning mode is not a substitute for the Fabric toggle. Preview in Plan,
review changes, obtain approval, apply only that plan in Act, then return to Plan.
If the UI or available integration cannot perform an operation, hand that step
to the participant rather than bypassing the supported authoring path.

Never delete resources, reset a shared folder, reassign capacity, overwrite data,
clear ownership ledgers, weaken checksum checks or silently migrate an existing
deployment. Stop and explain conflicts or uncertain writes.

## Guided sequence

### 1. Establish the starting point

Ask whether the participant is starting fresh or has already staged sources or
generated an ontology. Confirm their intended workspace and folder without
assuming a particular tenant, URL or folder name.

Check the applicable prerequisites in the walkthrough: supported active capacity,
Contributor or higher, workload permissions and ontology preview availability.
If you cannot inspect them directly, ask the participant to confirm them.
Do not change capacity or tenant settings yourself without separate approval.

### 2. Preview the published setup

For a fresh deployment, guide them to download and import
[Stage_Zava_Workshop.ipynb](Stage_Zava_Workshop.ipynb) as PySpark. Put it in the
chosen deployment folder before its first run. At workspace root, the workspace
must otherwise be empty; setup creates SupplyChain.

Do not attach a default Lakehouse. Use `APPLY = False`,
`CONFIRM_WORKSPACE_NAME = ""` and `REPO_REVISION = "main"` for preview.
After approval to run the preview, check the printed workspace, folder, source
names and resolved commit. Pin that full commit before apply so preview and
apply use identical files. Explain that data/model inputs are downloaded over
HTTPS and verified, not embedded in the notebook.

### 3. Apply and monitor setup

Explain the proposed creations: a 19-table Lakehouse, source Eventhouse/KQL
database with four telemetry tables, and a 13-table semantic model. No ontology
is installed. Ask for approval before setting `APPLY = True` and the exact
confirmed workspace name, or before asking the participant to run setup.

If using execution APIs, check for active jobs and runs from the last five
minutes first. Inspect recent jobs instead of submitting duplicates. Capture
the operation/job identity immediately after submission, then monitor that exact
operation. Never blindly retry create, ingest, refresh or job-start requests.

Wait for completion. Preserve the same notebook identity and location for
resumption. On failure, use the walkthrough's recovery guidance; an error or
unknown operation outcome is not permission to rerun writes.

### 4. Check the sources before moving on

Verify the exact newly staged items are in the intended folder. Wait for SQL
metadata synchronization; do not reingest to make tables appear.
Use the walkthrough's SQL and DAX checks, or guide the participant to run them.
The model smoke totals are 80 products, 100700 on-hand units, 72100 net available
units and 3534 at-risk order units. These are verification expectations, not
substitutes for actual query results.

The Lakehouse has 19 tables; the model deliberately has only 13. If SQL or DAX
access is unavailable, label the check blocked or participant-confirmed as
appropriate. Do not present notebook completion as proof of the entire lab.

### 5. Generate and inspect the participant ontology

After approval, guide the participant to generate a new ontology from their
staged semantic model and keep it in their deployment folder. Inspect the
actual generated entities, identifiers, relationships and metrics.

Keep the existing source model unchanged. Do not activate all inactive semantic
relationships. Reuse existing concepts rather than creating duplicate keys or
objects. If generation is unavailable, stop at that gate and report the feature
or permission blocker; do not substitute a prebuilt ontology.

### 6. Ask a simple business question

In Fabric **Plan**, start with:

> Where is our supply chain most vulnerable?

Follow an example with:

> Why is that a risk? Show me what depends on it.

If the current grounding cannot establish the answer:

> What additional data would help us answer this?

Treat this as a chance to explain the value of enrichment, not a reason to
infer dependencies that have not been modeled or queried.

### 7. Discover and review Lakehouse enrichment

Use the Lakehouse discovery prompt from the prompt card in **Plan**, scoped to
the participant's exact staged source. Review six proposed entities:
BillOfMaterial, SupplierSite, ComponentSupply, SupplyRoute, RouteHub and
LogisticsHub. Check their keys and 11 joins against the `tables` and
`relationships` sections of the
[dependency contract](data/dependencies/contract.json).

Reuse existing Product, Component, Supplier, Facility and Carrier entities.
Facility's key is ObjectId. Keep the six new tables outside the semantic model;
preserve existing Shipment/InventoryPosition grounding and semantic measures.
Do not feed the contract's scenario answers into the business conversation.

Ask for approval of the actual preview. Apply only the reviewed plan in **Act**,
return to **Plan**, and confirm that entities and bindings persisted.

### 8. Follow the discovered dependencies

Return to "Where is our supply chain most vulnerable?" Follow the answer:

> Why is that a risk? Show me what depends on it.

> Do we have a genuine alternative?

> Which customer orders could be affected?

Do not require a known hub, supplier, product, order or date in the prompt.
Check both primary-component links and bills of material; count each order once
across multiple paths. Compare shared dependencies, qualification, lead time and
capacity. Discover snapshot times and make any recovery-horizon assumption
explicit. LeadTimeHours already includes TransitHours.

Do not equate dependency exposure with lost sales, a shipment-delay cause or
proof that an alternative is feasible.

### 9. Review and apply operational observations

Use the telemetry enrichment prompt in **Plan**, selecting the participant's
source Eventhouse/KQL database, not an ontology-generated companion.

Review the two bindings: ShipmentTelemetry to existing Shipment through
ShipmentId, and InventoryPositionTelemetry to existing InventoryPosition
through InventoryPositionId. Both use ObservationTime for ordering.

Check all six properties: shipment delay, handling shock and route risk;
inventory available-to-promise units, days of supply and stockout risk.
Keep the static entities, relationships and semantic grounding unchanged.

Obtain approval, apply only the reviewed plan in **Act**, return to **Plan**
and check that the observations and bindings persisted.

### 10. Discover deliveries and compare responses

Start with:

> Which deliveries need our attention?

Follow the actual answer:

> What makes these stand out?

> Should we respond to all of them the same way?

> What have we already decided, and what still needs checking?

Discover the available observation range rather than using today's clock.
Select a bounded analysis window within the data and use the latest observation
per identity within it. Show selected times and freshness differences.
Missing observations are unknown, not zero risk; conflicting same-time values
need investigation. Do not force the walkthrough's example shipment pair.

Distinguish recorded Approved/Proposed decisions from actions actually executed.
Do not infer a universal quality policy from a shock value or claim available
stock proves route feasibility. No operational intervention is executed here.

### 11. Inspect evidence and finish honestly

At any point, ask:

> Show me the data behind that.

For the final explanation, request actual queries, sources and observation times.
Separate facts, recorded decisions, interpretations and open human checks.
If source traces or query execution are unavailable, mark that verification
incomplete even when the answer sounds plausible.

Finish with a short account of the completed gates, the participant's discovered
business example, its supporting evidence and any blockers or assumptions.
Graph materialization, sharing, permissions testing and operational action
execution are separate follow-on exercises, not implied successes.
