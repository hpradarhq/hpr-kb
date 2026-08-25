# BeltRisk — Data Explorer, Evidence & Override Governance

**Status:** ACTIVE  
**Maturity:** MEDIUM  
**Last Updated:** 2026-08-25

## 1. Intent

BeltRisk must remain simple at the decision surface without becoming a black box.

The canonical UX principle is:

> **Decision first → reason second → evidence on demand → controlled override when justified.**

Therefore the application keeps a full Data Browser / Explorer, but it is not the primary workflow for ordinary users. It is the evidence and governance layer behind recommendations.

The product must never require a Plant Director to manually inspect raw tables before receiving value. Equally, a reliability engineer, warehouse user, buyer, auditor, or plant leader must always be able to inspect the evidence that produced a recommendation.

---

## 2. Three Product Surfaces

### A. Decision Surface

Purpose: fast operational action.

Answers:

- What is exposed now?
- What can safely run with zero stock?
- What needs ordering, expediting, inspection, or replacement?
- What capital can be released without crossing the accepted risk boundary?

This surface hides most engine terminology.

### B. Evidence Explorer

Purpose: verify the facts and reasoning behind a recommendation.

A user can drill from an asset recommendation into the exact source evidence used by the engine.

Typical evidence domains:

- asset master and belt specification;
- installation / commissioning history;
- inspections and condition observations;
- wear measurements and RUL inputs;
- sudden-event history;
- operating exposure: hours, tonnes, starts, campaigns;
- stock quantity and stock fitness;
- alternate / shared spare compatibility;
- PO and supplier delivery milestones;
- supplier lead-time history;
- quality acceptance / rejection history;
- emergency recovery capability;
- downtime consequence assumptions;
- replacement and maintenance cost assumptions;
- engine rule trace and recommendation history.

The explorer is not merely a database viewer. It must preserve context: source, timestamp, owner, confidence, validity period, and whether a value is observed, imported, derived, assumed, or manually overridden.

### C. Override / Governance Surface

Purpose: allow qualified human judgment to supersede a derived recommendation or input when reality is better known than the model.

Overrides are first-class, explicit, time-bounded business facts — never silent edits to calculated output.

---

## 3. Data Provenance Is Mandatory

Every decision-driving value should carry provenance wherever practical.

Canonical metadata:

- `value`
- `source_type`
- `source_reference`
- `observed_at` or `effective_from`
- `effective_to` when applicable
- `entered_by` / `system_owner`
- `confidence`
- `verification_status`
- `override_status`
- `reason`

Useful source classes:

- `MEASURED`
- `ERP_IMPORTED`
- `WMS_IMPORTED`
- `PO_IMPORTED`
- `MANUAL_VERIFIED`
- `SUPPLIER_CONFIRMED`
- `DERIVED`
- `EXPERT_PRIOR`
- `OVERRIDE`

The UI may simplify these visually, but audit detail must retain them.

---

## 4. Explorer Design Rule

The explorer should answer:

> **“Show me exactly why BeltRisk believes this.”**

For example, from:

`26BC1 → ORDER JIT`

one click should expose the chain:

```text
Recommendation: ORDER JIT

Asset evidence
  condition stable
  wear trend low
  planned replacement 18 Nov

Sudden-event evidence
  1 cut / 8.2 Mt
  evidence confidence MEDIUM

Supply evidence
  stock 0
  supplier capability PROVEN
  historical P90 usable lead time 51d
  quality first-pass 11/11
  second source YES

Cost evidence
  holding spare now = X
  JIT policy avoids Y months inventory

Rule trace
  sudden-event guardrail = PASS
  lead-time guardrail = PASS
  protected arrival window = PASS

Decision
  ORDER JIT
```

The user should not need SQL knowledge to inspect this chain.

---

## 5. Raw Data Browser Still Matters

Qualified technical users should also have a lower-level browser for source records.

Recommended capabilities:

- filter by asset, supplier, specification, date, event type, PO, source system;
- inspect raw historical records;
- compare imported value vs normalized / derived value;
- view missing-data gaps;
- view stale data;
- view data conflicts;
- export evidence for offline analysis;
- trace a displayed metric back to source records;
- see engine calculation / rule version;
- see BE / FE build version relevant to a recommendation when required for audit.

This raw browser is an expert tool. It should not dominate the normal application navigation.

---

## 6. Human Override Principles

Human override is required because industrial reality contains information that systems may not yet know.

Examples:

- a spare shown as `stock = 1` is physically damaged and unusable;
- a supplier confirms a factory shutdown not yet represented in historical lead time;
- an emergency compatible belt is available from another plant;
- a planned shutdown has moved;
- an inspection found a serious cut that has not yet entered the normal data stream;
- purchasing knows a PO is commercially blocked despite a nominal confirmed status;
- a plant engineer knows a substitute specification is technically acceptable.

The system must allow this knowledge to enter explicitly.

### Hard rule

> **Override the fact or policy assumption, not the final answer without explanation.**

Preferred:

`Supplier lead-time planning value: 55d → override 90d until 2026-10-31; reason: factory maintenance shutdown; approved by X.`

Then recompute the recommendation.

Avoid:

`Recommendation ORDER JIT → manually change to HOLD SPARE`

with no underlying rationale.

A direct recommendation override may still be allowed for exceptional management decisions, but it must carry stronger governance and a mandatory reason.

---

## 7. Override Types

### Input Override

Changes a decision-driving fact for a defined validity period.

Examples:

- usable stock quantity;
- supplier planning lead time;
- shutdown date;
- compatibility status;
- emergency recovery readiness;
- consequence class.

The engine recomputes normally using the overridden fact.

### Policy Override

Changes a policy parameter.

Examples:

- force `SAFE` mode for one asset family;
- minimum physical spare = 1;
- do not rely on sole-source JIT for a critical route;
- temporary larger risk buffer during a campaign.

### Recommendation Override

Management explicitly chooses an action different from the engine recommendation.

This should be less common and require:

- user identity;
- reason;
- timestamp;
- expiry / review date;
- previous recommendation;
- selected action;
- optional approval depending on consequence.

---

## 8. Override Lifecycle

Every override should be:

1. **Explicit** — visible to users affected by it.
2. **Attributed** — who made it.
3. **Reasoned** — why.
4. **Time-bounded** — permanent overrides should be rare.
5. **Reviewable** — expiry or review date.
6. **Reversible** — original evidence is never destroyed.
7. **Auditable** — before/after values retained.
8. **Recomputed** — recommendation regenerated after input/policy changes.

Suggested states:

- `ACTIVE`
- `EXPIRING`
- `EXPIRED`
- `SUPERSEDED`
- `REVOKED`

---

## 9. Data Quality Must Be Visible

BeltRisk must not hide weak input behind a confident recommendation.

For every recommendation the engine should internally know whether its important evidence is:

- complete;
- stale;
- missing;
- conflicting;
- low-confidence;
- manually overridden.

Normal UI can collapse this to a simple signal such as:

`Evidence: STRONG / FAIR / WEAK`

But the explorer must explain why.

Example:

```text
Evidence FAIR

Asset condition      STRONG
Sudden-event history MEDIUM
Stock                STRONG
Supplier lead time   WEAK — only 2 observed deliveries
Cost assumptions     MEDIUM
```

Low data quality should widen buffers or demand verification rather than silently produce false precision.

---

## 10. Supply-Chain Evidence Browser

Supply is important enough to deserve its own explorer perspective.

Browse by `supplier × spare/spec family × route`, not supplier name alone.

Show:

- capability status: PROVEN / QUALIFIED / CLAIMED / UNKNOWN / NOT_CAPABLE;
- quoted lead time;
- observed P50 / P80 / P90 usable lead time;
- sample size;
- promise adherence;
- P90 promise slip;
- first-pass quality acceptance;
- QC release delay;
- second qualified source;
- logistics route dependency;
- emergency repair / response capability;
- active PO milestones;
- current disruptions / manual overrides;
- evidence confidence.

A single composite health score may be shown for convenience, but must never replace these underlying facts or allow one RED hard gate to be hidden inside an average.

---

## 11. Role-Oriented Usage

### Plant Director / Manager

Primary surface: Decision Board.

Explorer use: only when challenging a recommendation or reviewing material exposure.

### Reliability / Maintenance Engineer

Primary surface: Decision Board + Asset Evidence.

Explorer use: inspect condition, event history, compatibility, rule trace, and override technical assumptions.

### Warehouse / Spare Parts

Primary surface: coverage and stock truth.

Explorer use: verify physical stock, stock fitness, location, shared-spare applicability.

### Purchasing / Supply Chain

Primary surface: due procurement actions.

Explorer use: supplier/spec/route performance, PO milestones, lead-time evidence, quality and resilience.

### Auditor / Administrator

Primary surface: provenance and change history.

Explorer use: full rule trace, source data, override history, engine/build version.

---

## 12. UX Invariant

The existence of the Data Explorer must not cause the Decision Surface to become dense again.

Canonical layering:

```text
L0  DECISION
    What do I need to do?

L1  WHY
    The 3–5 facts that explain the action.

L2  EVIDENCE EXPLORER
    Source data, history, confidence, rule trace.

L3  RAW / ADMIN
    Full records, imports, correction, controlled override, audit.
```

The product should progressively disclose complexity rather than expose all layers at once.

---

## 13. Core Invariants

**CANONICAL**

1. A recommendation must be traceable to its inputs.
2. A displayed input must be traceable to its source where possible.
3. Derived values must be distinguishable from observed values.
4. Overrides never erase original evidence.
5. Overrides must have actor, reason, time and validity.
6. Input/policy overrides trigger recomputation.
7. Weak data must reduce confidence or increase protection; it must not create false certainty.
8. The Decision Surface stays simple even as the evidence model becomes richer.
9. Expert users retain a powerful browser/explorer for verification and investigation.
10. BeltRisk remains decision support; human authority can override when justified, but the disagreement must remain visible and auditable.

---

## 14. Related Streams

- `beltrisk-risk-constrained-jit.md`
- `plantops-platform.md`

## 15. Sources

- `raw_available: false`
- `source_status: context-derived`
- Canonicalized from BeltRisk V2 product discussion on 2026-08-25.
