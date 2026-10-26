# Zava prompt card

If setup deployed beside the notebook in a custom folder, replace **SupplyChain**
below with that folder's name.
Use the source names printed by setup, including `_Lab_...` suffixes. Keep the
conversation scoped to those newly staged sources, not similarly named items
elsewhere in the workspace.

Set this context once in your participant ontology conversation:

> We are working in SupplyChain on my Ignite2026 Zava workshop ontology.
> Keep the source data, semantic model and any reference ontology unchanged.
> Discover the available snapshot and observation times from the data rather
> than using today's clock. Show the dates, analysis window and assumptions
> behind time-sensitive answers.

## 1. Start with a business question

In **Plan**, start simply:

> Where is our supply chain most vulnerable?

Follow an example from the answer, rather than supplying an identifier:

> Why is that a risk? Show me what depends on it.

If the available model cannot establish the dependencies:

> What additional data would help us answer this?

The starting answer is limited to the generated ontology and its current
grounding. Acknowledge missing context; do not claim an exhaustive risk ranking.

Optional model review:

> Check the identifiers and relationships. Show anything that needs attention,
> but don't change anything.

## 2. Discover new Lakehouse entities

> Find the supply-chain dependencies in our Zava lakehouse that aren't in this
> ontology yet. Add the new business objects and connect them to what we already
> have. Keep the existing model and measures. Show the plan first.

If it reports no tables:

> What can you see in the Zava lakehouse?

For existing concepts that are already grounded:

> Show how shipments and inventory are already grounded. Don't add duplicates.

The expected discovery is six new entities, not six new semantic-model tables.
Review their own identifiers and the 11 connections before applying.

## 3. Apply the reviewed plan

Switch from **Plan** to **Act** only after checking the preview:

> Apply only the plan we just reviewed. Keep everything else unchanged.

**After apply, switch back to Plan.** Keep Plan selected for questions and all
subsequent authoring previews.

### Explore the dependencies

Return to the opening question after enrichment:

> Where is our supply chain most vulnerable?

Follow something it finds, one question at a time:

> Why is that a risk? Show me what depends on it.

> Do we have a genuine alternative?

> Which customer orders could be affected?

Keep the answer scoped to this modeled network. An independent path is not
automatically qualified, timely or feasible; exposure is not a lost-sales forecast.
Check qualification, lead time, capacity and shared dependencies before calling
an alternative usable, and state any recovery-horizon assumption. Trace both
primary-component links and bills of material, counting each order only once.
These are conversational follow-ups, not a rigid script; refer to the previous
answer rather than forcing a predetermined hub, product or supplier.

## 4. Plan telemetry enrichment

Confirm the toggle is **Plan**, not Act, before sending:

> Add shipment delay, handling shock and route risk, plus inventory availability,
> days of supply and stockout risk, from the Zava source Eventhouse. Match the
> existing objects and use observation time. Keep the model and measures.
> Show the plan first.

Review keys, table names, ordering timestamp and all six properties before using
the same apply prompt in Act. Return to **Plan** afterward. Include DaysOfSupply
in the inventory preview; if omitted, revise it while still in **Plan**:

> Include days of supply with the inventory observations in that plan.

## 5. Discover deliveries that need attention

In **Plan**, start with:

> Which deliveries need our attention?

Follow the deliveries identified in the answer:

> What makes these stand out?

> Should we respond to all of them the same way?

> What have we already decided, and what still needs checking?

Let the agent discover identifiers and observation times. Inspect the available
time range, choose a bounded window within the data and use the latest observations
per identity within it. Show the selected times and any freshness differences.
Missing observations are unknown, not zero risk. Distinguish recorded Approved
or Proposed decisions from actions actually executed.

## 6. Inspect the evidence

At any point, ask:

> Show me the data behind that.

For the final explanation:

> Show the actual queries, sources and observation times behind that answer.
> Separate recorded facts from your interpretation.

If the answer doesn't establish query execution:

> Which source queries actually ran? Show their bounded results.

No prompt contains the expected answer. The builder oracle belongs outside the
agent's grounding context. Do not combine preview and apply into a single prompt.
