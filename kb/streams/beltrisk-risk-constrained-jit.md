# BeltRisk — Risk-Constrained Just-In-Time Spare & Replacement Intelligence

**Status:** ACTIVE  
**Maturity:** MEDIUM  
**Last Updated:** 2026-08-25

## 1. Intent

BeltRisk exists to answer one operational question well:

> **For each critical conveyor belt, what is the lowest-cost action we can take now without accepting an unsafe or operationally unacceptable risk of production loss?**

The product is not a CMMS, EAM, inventory system, purchasing system, or generic reliability dashboard.

Its decision boundary is deliberately narrow:

> **Asset Performance → Asset Risk → Coverage → Asset Cost → Recommended Action**

The target operating philosophy is:

> **JUST-IN-TIME, lowest acceptable RISK.**

This means zero spare stock can be an excellent state when evidence and lead time support it. Conversely, a confirmed purchase order is not automatically protection when the asset can fail suddenly before the material physically arrives.

---

## 2. Five-Minute Onboarding Mental Model

A new user should be able to understand BeltRisk with these five questions only:

1. **How healthy is the belt?**  
   Is performance stable, degrading, or already impaired?

2. **What can make it fail?**  
   There are two fundamentally different risk families:
   - gradual degradation / wear-out;
   - sudden events such as scrap metal cutting the belt, impact, jam, splice failure, foreign object damage, or other abrupt events.

3. **If it fails or reaches replacement time, are we covered?**  
   Coverage may be:
   - physical fit-for-purpose spare on hand;
   - a confirmed inbound spare that will arrive early enough;
   - an approved substitute / emergency repair path;
   - or no credible coverage.

4. **What does each option cost?**  
   The system compares the economic burden of carrying inventory, ordering early, replacing early, expediting, and accepting exposure to downtime.

5. **What should we do now?**  
   The app should end with one clear operational recommendation, for example:
   - MONITOR;
   - ORDER JIT;
   - EXPEDITE;
   - HOLD PHYSICAL SPARE;
   - REPLACE NEXT SHUTDOWN;
   - INSPECT / VERIFY CONDITION.

If a user must understand more than these five questions before the first useful decision appears, the UI has failed even if the underlying model is correct.

---

## 3. Executive Summary for Plant Leadership

BeltRisk does **not** try to minimize inventory alone and does **not** try to minimize technical risk alone.

Both extremes are economically poor:

- **Maximum stock / minimum risk:** large working capital, aging spares, storage burden, obsolete specifications, and hidden waste.
- **Minimum stock / maximum JIT:** attractive working capital but unacceptable exposure if lead times slip or sudden failures occur.

BeltRisk searches for the narrow region between them:

> **the minimum inventory and earliest economically justified action that still protects production against the risk actually supported by evidence.**

A good BeltRisk decision may therefore look counter-intuitive:

- `stock = 0` can be **GREEN** when a healthy asset has low sudden-event exposure and a confirmed spare will arrive before the protected replacement window;
- `stock = 0 + PO confirmed` can still be **RED** when sudden-event exposure is high because a PO is not a physical spare;
- an old belt may remain in service if its measured performance is stable and replacement can safely wait for the next planned shutdown;
- a visually healthy belt may still require physical protection if site history shows meaningful sudden-event frequency and downtime consequence is large.

---

## 4. Current Canonical State

### 4.1 Product North Star

**CANONICAL**

> **Asset Performance → Asset Risk → Asset Cost**

For operational decisioning this is expanded to:

> **Performance → Risk → Coverage → Cost → Action**

`Coverage` is made explicit because it is the bridge between technical risk and procurement/inventory action.

### 4.2 V2 Engineering Direction

**CANONICAL**

V2 implements a deterministic risk-constrained JIT engine with:

- degradation / wear risk;
- sudden-event historical risk;
- replacement timing;
- stock and inbound coverage;
- procurement lead time;
- consequence / downtime sensitivity;
- policy optimization from conservative to aggressive inventory reduction;
- auditable recommended actions.

### 4.3 Current UX Assessment

**CANONICAL**

The V2 engine is useful, but the current V2 user interface should be treated as an **engineering truth surface**, not as the final plant-leadership interface.

The current design exposes too many internal concepts at once. A plant director or maintenance leader should not need to reverse-engineer the model from badges, states, confidence labels, risk categories, lead-time values, and cost terms.

The final UI must follow:

> **Decision first → reason second → evidence on demand.**

The complexity belongs in the engine and audit layer, not on the first screen.

---

## 5. Risk Model: Two Different Failure Worlds

### 5.1 Degradation / Wear-Out Risk

This is the predictable side of belt life.

Typical evidence:

- belt age / service duration;
- thickness or wear observations;
- splice condition;
- tracking problems;
- repeated local repair;
- throughput / loading history;
- RUL estimate;
- planned replacement date;
- trend deterioration.

This risk normally changes gradually and supports planned replacement.

The main question is:

> **How long can the asset remain in service before the probability or consequence of degradation becomes unacceptable?**

For this risk family, JIT procurement can work very well because there is usually a planning horizon.

### 5.2 Sudden-Event Risk

This is the abrupt side of belt failure.

Examples:

- scrap metal cutting the belt;
- foreign material impact;
- chute blockage / jam;
- sudden splice separation;
- structural interference;
- abnormal object ingestion;
- severe mis-tracking event;
- other discrete accidental damage.

A belt may have excellent remaining wear life and still be exposed to this class of event.

The model therefore **must not derive all risk from wear or RUL**.

Sudden-event risk is learned from history using operational exposure rather than raw event count alone. The intent is not to invent precise probabilities from poor data, but to distinguish cases such as:

- no meaningful evidence of sudden failures;
- limited evidence / low confidence;
- recurrent site-specific exposure;
- high-consequence recurrent exposure.

The result is used as a guardrail on how aggressively inventory can be squeezed.

### 5.3 Evidence Confidence

The system must distinguish **risk level** from **confidence in that risk estimate**.

Examples:

- zero recorded failures over a long, trustworthy history may support low sudden-event risk;
- zero failures in a new or poorly logged dataset does **not** justify the same conclusion;
- several events in a high-exposure asset fleet may mean something different from the same count on one lightly used belt.

Low confidence should generally widen the protection buffer rather than create false precision.

---

## 6. Coverage Model

Risk alone does not determine action. The next question is whether the plant can recover if the asset reaches its replacement point or fails unexpectedly.

### 6.1 Physical Coverage

Strongest protection:

- correct belt / specification physically available;
- usable condition confirmed;
- accessible when required;
- installation capability and shutdown path available.

### 6.2 Inbound JIT Coverage

A confirmed order may count as JIT coverage only when it can arrive before the protected need window **and** sudden-event exposure is sufficiently low.

This is the key distinction:

> **A confirmed PO is a time promise, not a physical spare.**

Therefore:

`healthy asset + low sudden-event risk + planned replacement + confirmed ETA before protected window + stock 0`

can be classified as:

> **JIT_COVERED — zero stock is intentional and economically good.**

But:

`high sudden-event risk + stock 0 + PO confirmed`

must remain:

> **INBOUND_GAP — production is still physically exposed until the spare arrives or another credible temporary coverage exists.**

### 6.3 Alternative / Temporary Coverage

Depending on asset and site practice, temporary coverage may include:

- approved compatible spare;
- repair kit / hot or cold splice capability;
- temporary belt section;
- bypass / alternative material route;
- rapid vendor emergency service;
- other validated recovery path.

These should never be assumed. They count only when they are operationally credible and auditable.

---

## 7. The JIT Timing Logic

BeltRisk should think in **protected time windows**, not simply `stock yes/no`.

For each asset there are four important times:

1. **Expected need date** — when replacement is likely or planned.
2. **Procurement lead time** — realistic time from order to usable arrival, not nominal catalogue lead time.
3. **Risk buffer** — extra protection time required because of uncertainty, sudden-event exposure, consequence, supplier reliability, and evidence quality.
4. **Arrival date / physical coverage date** — when the spare becomes actually usable at the plant.

The logic is conceptually:

> **Latest safe procurement point = expected need date − realistic lead time − required risk buffer.**

The exact numerical implementation may evolve, but the business meaning must remain stable.

A procurement plan is acceptable only if physical coverage arrives before the protected window closes.

---

## 8. Optimization / Squeeze Policy

The V2 slider represents **risk appetite for working-capital optimization**, not permission to violate safety or production guardrails.

The conceptual policy modes are:

| Mode | Meaning | Typical behavior |
|---|---|---|
| **SAFE** | Conservative protection | Larger buffers; more physical spares retained |
| **BALANCED** | Normal industrial policy | JIT where evidence is good; stock where consequence/risk justifies it |
| **LEAN** | Strong working-capital pressure | Smaller buffers; greater reliance on reliable inbound coverage |
| **SQUEEZE** | Minimum credible inventory | Push stock toward zero wherever hard risk guardrails still permit it |

### Hard invariant

> **The slider may change economic preference and buffer size, but it may never override a hard risk guardrail.**

Examples of guardrails that can block squeezing:

- high sudden-event exposure;
- no credible recovery method;
- critical production consequence;
- unreliable or highly variable supplier lead time;
- low-quality evidence combined with severe consequence;
- replacement window too close for inbound supply;
- incorrect / uncertain spare specification.

The continuous numerical slider can remain an internal engine parameter. The normal user-facing control should strongly prefer the four named modes because they are understandable and auditable.

---

## 9. Worked Example — 26BC1

Assume:

- asset: `26BC1`;
- replacement is planned rather than reactive;
- current physical stock: `0`;
- procurement can be arranged just in time.

### Case A — Good JIT candidate

Evidence shows:

- stable performance;
- sufficient remaining life;
- low sudden-event history with adequate confidence;
- known specification;
- supplier lead time is credible;
- confirmed arrival is before the protected replacement window.

Decision:

> **Do not hold a spare unnecessarily. Order JIT and preserve working capital.**

Here `stock = 0` is not a deficiency. It is the desired optimized state.

### Case B — Same planned replacement, but meaningful sudden-event exposure

Evidence shows recurrent belt cuts or abrupt failures from foreign material.

Even if:

- planned replacement is months away; and
- a PO is confirmed,

there is still a period during which an abrupt failure would leave the plant without physical recovery coverage.

Decision:

> **Do not classify the asset as covered merely because a PO exists. Maintain / accelerate physical coverage or establish a credible temporary recovery path.**

This is why sudden-event history materially changes the JIT decision.

---

## 10. Cost Logic

BeltRisk should compare **decision costs**, not merely purchase price.

Relevant economic components include:

- purchase price;
- inventory carrying cost;
- storage / preservation cost;
- obsolescence or wrong-spec risk;
- premium freight / expedite cost;
- early replacement cost / unused life sacrificed;
- planned shutdown replacement cost;
- unplanned downtime consequence;
- secondary damage / cleanup / production disruption where relevant.

The model should avoid fake precision. If the business cannot credibly quantify a component, the UI should show the assumption or use a bounded scenario rather than fabricate a precise number.

The product question is not:

> “Which option has the lowest purchase cost?”

It is:

> **“Which option has the lowest lifecycle decision cost while staying inside the acceptable operational risk boundary?”**

---

## 11. Deterministic and Auditable by Design

**CANONICAL**

BeltRisk recommendations must be reproducible from explicit inputs and rules.

The system should be able to explain every recommendation in plain language:

> “ORDER JIT because the belt is stable, replacement is planned for the next shutdown, sudden-event evidence is low, supplier lead time plus protection buffer fits inside the remaining window, and holding stock now adds carrying cost without materially reducing risk.”

or:

> “HOLD / EXPEDITE PHYSICAL SPARE because sudden-event history is material and a confirmed PO does not protect the plant before arrival.”

This is more important than producing sophisticated but opaque probabilities.

---

## 12. Safety and Control Boundary

BeltRisk is decision support.

It does **not** control PLC/DCS equipment and does not issue automatic process commands.

Safety and actual equipment state remain separate from prediction and economic optimization.

Canonical internal separation:

1. **Actual Operating State** — what the equipment is doing now.
2. **Predicted Risk** — what may happen based on condition/history.
3. **Recovery Readiness / Coverage** — what protection exists if something happens.
4. **Economic Decision** — what action is justified given risk and cost.

These are useful engine concepts but should not all be presented as equal-weight top-level UI objects.

---

## 13. Why the Current V2 UI Is Too Hard

The current V2 UI reflects the structure of the reasoning engine too directly.

That creates several cognitive problems:

- too many simultaneous dimensions;
- internal terminology appears before the decision;
- users must mentally combine wear risk, sudden-event risk, coverage, lead time, consequence, cost, and confidence;
- risk categories and supporting evidence compete visually with the recommended action;
- the slider is understandable conceptually, but the surrounding decision surface is still too dense;
- a senior plant leader is forced into analyst mode before receiving executive value.

This is an architectural UX mistake, not a flaw in the underlying decision model.

### UX invariant

> **A user must never need to understand the engine in order to use the recommendation.**

The engine may be complex. The first screen must not be.

---

## 14. Target Information Architecture

The correct UI is three layers deep.

### Layer 0 — 30-Second Plant Decision Board

The opening screen should answer only:

- **Where are we exposed now?**
- **Where can we safely reduce stock?**
- **What actions are due next?**

Recommended top-level groups:

#### DO NOW
Assets with uncovered or unacceptable exposure.

Examples:

- expedite spare;
- inspect immediately;
- hold physical spare;
- replace next available shutdown.

#### PLAN
Assets with a known future action and adequate current protection.

Examples:

- order by date X;
- replace in shutdown Y;
- verify condition in N days.

#### SAFE TO SQUEEZE
Assets where physical inventory can be reduced or kept at zero without crossing the risk boundary.

A director should understand the plant position from this layer without opening a single detail panel.

### Layer 1 — Asset Decision Card

One card should contain approximately five things:

1. `26BC1`
2. **Decision:** `ORDER JIT — no stock required now`
3. **Risk:** `Wear LOW · Sudden LOW`
4. **Coverage:** `PO required by 15 Oct / arrival before protected window`
5. **Value:** `Avoid carrying one spare for ~N months` or another simple cost statement

Then one sentence:

> **Why:** healthy belt + low shock history + reliable lead time + planned shutdown.

No additional ontology is required for the normal decision.

### Layer 2 — Evidence / Audit Drawer

Only when the user asks “why?” should the app reveal:

- condition history;
- RUL logic;
- sudden-event history and exposure normalization;
- evidence confidence;
- procurement lead-time evidence;
- stock / PO details;
- consequence assumptions;
- cost breakdown;
- rule trace;
- recommendation history.

This is where engineering richness belongs.

---

## 15. Language Rules for the UI

Prefer operational language over model language.

| Internal / analytical concept | Preferred user-facing wording |
|---|---|
| Predicted Interlock Risk | Failure risk / production risk |
| Recovery Readiness | Spare coverage / recovery coverage |
| Sudden-event normalized exposure | Sudden failure history |
| RUL confidence | Confidence in remaining-life estimate |
| JIT_COVERED | JIT covered |
| INBOUND_GAP | Spare not physically covered yet |
| Optimization parameter 0–100 | SAFE / BALANCED / LEAN / SQUEEZE |

A senior user should be able to read the page without training in reliability statistics or the app's internal ontology.

---

## 16. Minimum Data Needed to Start

BeltRisk should be useful before the plant has a perfect sensor or reliability dataset.

Minimum practical onboarding data:

1. critical conveyor / belt asset list;
2. belt specification / compatibility;
3. current installed belt and approximate service start;
4. current spare stock;
5. planned replacement / shutdown information;
6. supplier and realistic lead time;
7. open purchase orders and expected arrival;
8. historical failure / belt-cut / splice / major repair events;
9. simple consequence class for production loss.

Additional condition data improves confidence but should not be a prerequisite to initial value.

Missing data should reduce confidence and widen risk buffers rather than make the application unusable.

---

## 17. Business Rules / Invariants

1. **Risk and stock are not the same thing.** High stock does not make a bad asset healthy; low stock does not automatically mean high risk.
2. **A PO is not physical coverage.** It becomes relevant only through its credible arrival date and the risk during the gap.
3. **Sudden-event risk is independent of wear-out risk.** A healthy belt can still need protection.
4. **Zero stock can be a positive optimized state.** The UI must explicitly celebrate safe JIT, not show every zero as a shortage.
5. **SQUEEZE cannot override hard guardrails.** Optimization ends where unacceptable risk begins.
6. **Evidence quality matters.** Missing or weak history must not be silently interpreted as low risk.
7. **Recommendations must be deterministic and explainable.** Same inputs + same policy must produce the same recommendation.
8. **Decision first, evidence second.** Normal users should never have to reconstruct the recommendation manually.
9. **BeltRisk does not control the plant.** It supports maintenance/procurement/operations decisions; PLC/DCS control remains separate.
10. **Avoid generic EAM/CMMS expansion.** Every new feature must strengthen Performance → Risk → Coverage → Cost → Action.

---

## 18. What the App Is Not

BeltRisk should reject feature gravity toward:

- generic work-order management;
- complete spare-parts warehouse management;
- purchasing workflow;
- ERP replacement;
- broad asset registry;
- generic dashboarding;
- alarm management;
- PLC control;
- black-box predictive-AI scoring with no audit trail.

Integration with these domains is valid. Rebuilding them inside BeltRisk is not.

---

## 19. Product Test: Can a Plant Director Use It in Five Minutes?

Before calling a UI revision complete, run this test with no explanation from the developer.

Within five minutes, a plant director should be able to answer:

1. Which belts threaten production now?
2. Which belts have enough protection?
3. Which spares can safely remain at zero?
4. What must Purchasing order or expedite next?
5. What is the approximate money tied up or released by the policy?
6. Why did the system make one selected recommendation?

If the user instead spends those five minutes learning status taxonomies, confidence scales, event-rate math, or internal engine terminology, the UI is not ready.

---

## 20. Recommended Next UX Revision

**TENTATIVE**

Do not rewrite the V2 engine. Flatten the presentation.

Target next surface:

### Header

`BeltRisk · Plant exposure · Spare capital · Next required action`

### Three plant-level numbers

- `Uncovered critical assets`
- `Spare capital currently held`
- `Capital safely releasable / avoidable`

### Three decision lanes

- `DO NOW`
- `PLAN`
- `SAFE TO SQUEEZE`

### Per-asset row/card

`Asset → decision → why → deadline → value`

Everything else moves behind `Why?` / `Evidence`.

The SAFE/BALANCED/LEAN/SQUEEZE policy control remains useful, but it should change the board outcomes rather than force the user to interpret the policy mathematics.

---

## 21. Open Questions

1. What is the best plant-level economic KPI: spare capital avoided, risk-adjusted cost, or both?
2. Which consequence categories are sufficient for V1/V2 without creating false precision?
3. How should supplier lead-time reliability be learned from history when procurement data is sparse?
4. What temporary recovery methods are credible enough to count as coverage for each belt class?
5. Should sudden-event history be normalized by operating hours, tonnes conveyed, starts, or another exposure denominator per asset family?
6. What minimum evidence threshold should be required before SAFE TO SQUEEZE is shown without a warning?

---

## 22. Pending Actions

1. Redesign the V2 plant screen into the three-layer information architecture above.
2. Preserve the current deterministic engine and move most model detail into an evidence drawer.
3. Make safe zero-stock states visually positive rather than shortage-like.
4. Make uncovered inbound-gap states explicit when sudden-event risk is material.
5. Add a five-minute plant-director usability gate to release acceptance criteria.
6. Keep BE/FE immutable build tags visible but visually subordinate to the operational decision surface.

---

## 23. Related Streams

- [PlantOps Platform](plantops-platform.md)
- [GitHub–Tailscale Immutable Artifact Deployment](github-tailscale-artifact-deployment.md)

---

## 24. Evolution

### V1 — Asset Cost Foundation

Established the basic asset and lifecycle-cost foundation, but did not yet express the desired operational decision model strongly enough.

### V2 — Risk-Constrained JIT

Added the canonical direction:

- JUST-IN-TIME inventory;
- lowest acceptable risk;
- degradation and sudden-event risk treated separately;
- historical sudden events influence spare strategy;
- planned replacement can intentionally operate with stock = 0;
- confirmed inbound supply is not treated as physical protection during a high sudden-risk gap;
- optimization policy from SAFE to SQUEEZE with hard guardrails.

### UX lesson after V2 live

The engine became more rigorous faster than the UI became simpler. The next product step is therefore **not more analytical features**. It is to compress the reasoning into a director-grade decision surface while keeping full evidence available one level deeper.

---

## 25. Sources

- `source_status: context-derived`
- `raw_available: false`
- Canonicalized from BeltRisk V1/V2 design, implementation, deployment verification, and product review discussion on 2026-08-25.
