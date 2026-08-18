# HPR Atlas

## Intent

Define the canonical product, interaction, entity, rendering and frontend architecture for HPR Atlas: a unified live spatial tracker for aircraft, vessels, maritime AtoN and sensing stations, with restrained progressive disclosure and a shared visual language across the global Atlas and local HPR Edge surfaces.

## Current Canonical State

HPR Atlas should continue from the **V4.8 progressive-surfaces baseline**. The V4.9 helicopter/personality experiment is explicitly rejected as a stable baseline because it created class/layer collisions and separated helicopter body/rotor rendering in ways that broke visual coherence.

The next stable work should remain in the V4.8 line (for example `V4.8.3`) until composition discipline and native helicopter integration are verified. The frontend is still in visual/interaction refinement; backend contract wiring is deliberately later.

The four entity classes are fixed:

1. Aircraft
2. Vessel
3. AtoN
4. Station

Helicopter is not a fifth entity class. It is an Aircraft with rotorcraft-specific classification and rendering.

The strongest composition rule from this session is:

```text
ONE PRIMARY CLASS AT A TIME

Aircraft
OR Vessel
OR AtoN
OR Station
```

Other classes may provide restrained contextual information but must yield foreground ownership to the active class.

## Confirmed Facts

- Repository: `ngtrthanh/hpr-atlas`.
- Canonical FE direction before this session: V4.8 progressive surfaces (`landing`, `atlas`, `edge`).
- A V4.9 helicopter/personality prototype was produced and then explicitly rejected by the user as visually chaotic.
- The user identified two concrete V4.9 failures:
  - entity/classes and overlays visually competed instead of yielding to one another;
  - helicopter bodies and rotors were rendered in mismatched coordinate systems, producing body/rotor separation.
- Helicopter SVG artwork is now supplied by the user in `hpr-atlas/main/talk`.
- Source artwork commit supplied in this session: `82b5312fed5622564d73dc75873535badd2e5b6a`.
- Known assets in `talk/` at that commit:
  - `Augusta_A129.svg`
  - `Bell_AH-1J.svg`
  - `Bell_AH-1Z_Viper.svg`
  - `V22.svg`
  - `h4.svg`
  - `xBell_222_1.svg`
  - `xBoeing_MH-6_1.svg`
  - `hand-off.md`
- The user will continue to optimize and add helicopter SVGs; Atlas should consume them rather than redraw them.
- The supplied V22 SVG exposes independent left/right rotor groups (`L-rotor`, `R-rotor`) and hubs, with native animation logic.
- Supplied Bell 222 and MH-6 SVGs also contain internal animation/group logic.
- A standalone demo that referenced `./talk/*.svg` failed outside the repository tree because the relative assets were unavailable.
- A later self-contained demo approach embedded the actual SVG content and successfully showed real helicopter artwork.

## Requirements

### Product / composition

- Keep Atlas map-first, restrained, premium and progressively disclosed.
- Preserve three V4.8 surfaces:
  - `landing` — quiet first contact.
  - `atlas` — global tracker.
  - `edge` — station-centric local surface.
- Do not turn the root experience into a corporate dashboard/card wall.
- Preserve distinct desktop, tablet and mobile compositions rather than scaling desktop down.
- Desktop: left entity/list rail, central map, right selected-entity detail without covering map controls.
- Mobile/tablet: only one sheet/panel should own interaction space at a time.

### Entity arbitration

- Exactly one primary entity class owns foreground at a time.
- Left/list belongs to the active class only.
- Right/detail belongs to the selected entity only.
- The map foreground belongs to the active class.
- Overlays are contextual/temporary and must not compete with primary entities.
- Do not reintroduce four independent entity-class checkboxes under Layers.

### Helicopter rendering

- Use user-supplied SVGs as authoritative artwork.
- Do not redraw, simplify or approximate helicopter silhouettes in Atlas code.
- One helicopter must remain one SVG coordinate system.
- Heading rotates the whole helicopter SVG.
- Rotor animation acts only on rotor groups inside that SVG.
- Never render body as one map symbol and rotor as a separate overlay.
- Preserve native/internal rotor animation when practical.
- Respect `prefers-reduced-motion` and pause/freeze animation when appropriate.

### Classification

- Do not classify helicopter only from ADS-B emitter category `A7`.
- Prefer exact type-code/database classification; use `A7` as fallback.
- Exact model artwork should be used only for an exact model match.
- A clearly wrong model must never be used as a fallback merely because it is visually attractive.

## Constraints

- The current exact SVG library does not yet cover the most common live helicopter types in the supplied ADS-B snapshot.
- Some standalone artifact/preview environments cannot resolve repository-relative `./talk` paths.
- SVGs have different source dimensions/viewBoxes and different amounts of whitespace, so raw canvas width/height is not a reliable visual-size metric.
- Helicopter rendering must not destabilize the four-class composition or become a separate architecture.
- Tar1090 is GPL; HPRadar may learn classification/registry concepts but should not casually import its GPL artwork/code into the public FE.

## Decisions

### Return to V4.8 as the stable frontend baseline

- Status: CANONICAL
- Decision: Do not patch the failed V4.9 forward; continue from V4.8 and make the next stable patch in the V4.8 line.
- Reason: V4.9 broke composition discipline and helicopter rendering coherence.
- Supersedes: Treating V4.9 as the new stable baseline.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

### Enforce one primary entity class at a time

- Status: CANONICAL
- Decision: Aircraft, Vessel, AtoN and Station do not simultaneously compete for foreground ownership.
- Reason: Multi-class visual competition created a cluttered, incoherent tracker.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

### Helicopter remains an Aircraft subtype

- Status: CANONICAL
- Decision: Rotorcraft/helicopter is represented under Aircraft, not as a fifth top-level ontology class.
- Reason: It is an airframe/rendering specialization, not a new spatial entity kind.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

### User-supplied SVGs are authoritative helicopter artwork

- Status: CANONICAL
- Decision: Atlas consumes the SVG files supplied under `hpr-atlas/talk`; frontend code must not redraw those aircraft.
- Reason: Exact, optimized model artwork should be separated from renderer logic and maintained as an asset library.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived; GitHub commit `82b5312fed5622564d73dc75873535badd2e5b6a`.

### Keep body and rotor in the same SVG coordinate system

- Status: CANONICAL
- Decision: A helicopter marker is one SVG object; airframe heading rotates the object and rotor groups animate internally.
- Reason: Separate body/rotor layers caused visible displacement and transform errors.
- Supersedes: Separate MapLibre body plus independent DOM rotor overlays.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

### Size helicopter icons by visual mass, not source canvas dimensions

- Status: CANONICAL
- Decision: Normalize displayed helicopter size visually against existing Atlas/HPR Marine markers; do not trust SVG canvas dimensions alone.
- Reason: Different SVG viewBoxes/whitespace made equal CSS pixel sizes look radically different.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

### Exact model match first; category/family fallback second

- Status: CANONICAL
- Decision: Rotorcraft detection and asset selection should prioritize exact type code/database knowledge, then family/generic rotorcraft fallback, with ADS-B `A7` only as one signal.
- Reason: The snapshot contained more known helicopter type codes than aircraft reporting category A7.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

### Learn from tar1090 architecture, not its public-facing artwork implementation

- Status: CANONICAL
- Decision: Reuse concepts such as type-designator precedence, registry separation, per-type scale normalization and lazy enrichment, while keeping HPRadar artwork/code independent.
- Reason: These are useful architecture patterns; direct reuse creates licensing and product-identity concerns.
- Source: ChatGPT session `hpr-atlas-heli-2026-08-18`, context-derived.

## Architecture / Model

### Core entity model

```text
HPR Atlas
├── Aircraft
│   ├── fixed-wing
│   └── rotorcraft
├── Vessel
├── AtoN
└── Station
```

Weather/meteo/hydro/tide remain overlays/observations, not a fifth entity class.

### Foreground arbitration

```text
selected entity
      ↓
active primary class
      ↓
context / relationships
      ↓
MLAT / coverage / weather overlays
      ↓
base map
```

### Helicopter classification

```text
exact type code / aircraft DB
        ↓
known rotorcraft mapping
        ↓
ADS-B category A7 fallback
        ↓
generic rotorcraft fallback
```

Conceptual runtime object:

```json
{
  "kind": "aircraft",
  "type_code": "B407",
  "category": "A7",
  "airframe_class": "rotorcraft",
  "render_profile": "heli-light-single"
}
```

`render_profile` can initially be computed in FE; it need not immediately become a backend contract field.

### Helicopter asset selection

```text
Aircraft
  ↓
isRotorcraft?
  ↓
exact asset available?
  ├── yes → use exact SVG from talk/
  └── no  → use geometry-family or generic rotorcraft asset
```

### Helicopter transform discipline

```text
SVG object
├── body / airframe groups
├── main rotor group(s)
├── tail rotor group (if any)
└── hubs / nacelles

whole SVG transform = map heading
internal rotor transforms = rotor animation only
```

### Rendering budget

```text
low zoom      → static/minimal rotorcraft representation
mid zoom      → selected/nearby rotorcraft animated
high zoom     → visible rotorcraft may animate
detail view   → full native animation
background tab → pause/freeze
```

This is a policy direction, not yet a measured hard performance cap.

## Business Rules / Invariants

- `live` does not mean `moving`.
- Helicopter remains `kind=aircraft` regardless of rendering personality.
- A wrong exact model silhouette is worse than a correct generic rotorcraft fallback.
- Renderer code should not own airframe artwork geometry.
- Asset library growth should not require renderer-architecture changes.
- The active class owns the foreground; inactive classes yield.
- Heli animation is personality, not a reason to weaken Atlas seriousness or map readability.
- Rotor RPM should not be fabricated as a direct function of groundspeed.

## Important Entities

- `ngtrthanh/hpr-atlas`
- `talk/` helicopter asset directory
- HPR Atlas V4.8
- HPR Edge surface
- Aircraft
- Vessel
- AtoN
- Station
- rotorcraft / helicopter
- MapLibre
- HPR Marine
- tar1090

## Important Numbers / Parameters

### ADS-B snapshot supplied in session

| Parameter | Value | Context |
|---|---:|---|
| Total aircraft tracked | 17,313 | User-provided ADS-B snapshot |
| Aircraft reporting category A7 | 242 | Rotorcraft emitter category |
| Approx. known heli type-code matches | ~310 | More than A7 count, proving A7 is incomplete |
| Total categorized aircraft | 11,587 | Snapshot category denominator |
| Total position-bearing aircraft | 16,785 | Snapshot |

### Common helicopter types in snapshot

| Type | Count |
|---|---:|
| AS50 | 53 |
| EC45 | 39 |
| B407 | 38 |
| EC35 | 37 |
| EC30 | 25 |
| R44 | 23 |
| B429 | 18 |
| H60 | 17 |
| R22 | 15 |
| A139 | 15 |
| A109 | 12 |
| R66 | 10 |

### Current sizing direction

| Surface | Approximate visual target |
|---|---:|
| Normal helicopter map marker | 64–72 px |
| Selected helicopter | 80–90 px |
| Helicopter in list | 36–40 px |
| Helicopter detail hero | 176–184 px |

These are tuning targets, not backend/business contract values.

## Rejected / Superseded Alternatives

- **Alternative:** Continue from V4.9.  
  **Why rejected/superseded:** Visual class collisions and broken body/rotor composition.  
  **Replaced by:** V4.8-based stabilization.

- **Alternative:** Show Aircraft, Vessel, AtoN and Station as equally dominant layers simultaneously.  
  **Why rejected/superseded:** No clear foreground ownership; visual overload.  
  **Replaced by:** One-primary-class arbitration.

- **Alternative:** Draw new generic helicopter vectors in FE code.  
  **Why rejected/superseded:** User has a dedicated optimized asset library and explicitly rejected ad-hoc redraws.  
  **Replaced by:** User-supplied SVG assets under `talk/`.

- **Alternative:** Render helicopter body and rotor independently.  
  **Why rejected/superseded:** Transform mismatch caused visible separation.  
  **Replaced by:** One SVG coordinate system.

- **Alternative:** Detect helicopter only when `category == A7`.  
  **Why rejected/superseded:** Snapshot demonstrated known heli types exceed A7-reporting aircraft.  
  **Replaced by:** Type-code/database classification plus A7 fallback.

- **Alternative:** Use one visually specific model (for example AH-1Z or CH-53) as a generic fallback for unrelated rotorcraft.  
  **Why rejected/superseded:** Visually attractive but factually wrong.  
  **Replaced by:** Family/generic rotorcraft fallback.

- **Alternative:** Depend on `./talk/*.svg` paths inside a standalone exported HTML demo.  
  **Why rejected/superseded:** Assets fail outside the repository tree.  
  **Replaced by:** Embed actual SVG content for standalone demos; repository-relative paths remain acceptable when running inside Atlas itself.

## Open Questions

- Final production location/name for the helicopter asset directory; `talk/` is current user-supplied location, but may later be promoted to a formal asset path.
- Exact mapping table from all live ICAO helicopter type codes to supplied model assets/fallback families.
- Final visual-size curve by zoom after testing against real Atlas aircraft icons.
- Whether internal SVG animation should remain asset-local in production or be normalized under an Atlas animation controller while preserving the same groups/pivots.
- Measured animation budget before choosing any hard maximum number of simultaneously animated rotorcraft.

## Pending Actions

- **NEXT:** Continue from V4.8 and produce a stable V4.8.x patch rather than resurrecting V4.9.
- **NEXT:** Inspect all current `talk/` SVGs and build explicit type-code → asset → rotor-group metadata.
- **NEXT:** Demonstrate at least four clear helicopter examples inside the tracker: MH-6, Bell 222, V22 and one attack helicopter such as AH-1Z/A129.
- **NEXT:** Verify Aircraft → Vessel → AtoN → Station switching with no residual foreground/layer collision.
- **NEXT:** Test desktop/tablet/mobile and dark/light profiles.
- **NEXT:** Test selected and unselected helicopter rendering, including V22 separately.
- **NEXT:** Verify no JavaScript errors and respect reduced-motion/background-tab behavior.
- **LATER:** Add exact-model assets for the most frequent live helicopter types, prioritizing AS50, EC45/H145, B407, EC35/H135, R44, B429, H60 and AW139/A139.

## Related Streams

- [HPRadarHQ](hpradarhq.md) — source-governance organization for the HPRadar product family.

## Evolution

The session began with the assumption that V4.9 would be a rotary-wing personality pass. Live ADS-B statistics then showed that rotorcraft classification had to be broader than `A7` and exact-model artwork could cover only part of the fleet. Tar1090 was reviewed as an architecture reference for type precedence, shape registries and scale normalization.

The first V4.9 implementation then failed visual review. The user rejected it because multiple classes/layers competed and helicopter rotor/body rendering was spatially incoherent. This caused a deliberate rollback to V4.8 and established stronger composition invariants.

The user then uploaded optimized helicopter SVGs to `hpr-atlas/main/talk` and explicitly made those assets authoritative. Subsequent work shifted from generating artwork to consuming, sizing and animating those SVGs correctly. Small-icon tests were rejected; visual targets were increased toward roughly 64–72 px on map and ~176–184 px in detail. Standalone demos also exposed an asset-loading constraint: repository-relative SVGs cannot be assumed to exist in exported artifacts.

The current direction is therefore not “V4.9 personality” but **V4.8 stabilization + native rotorcraft assets + strict class arbitration**.

## Sources

- ChatGPT session `hpr-atlas-heli-2026-08-18`
  - source_status: context-derived
  - raw_available: false
- `ngtrthanh/hpr-atlas` commit `82b5312fed5622564d73dc75873535badd2e5b6a`
  - user-supplied helicopter SVG assets under `talk/`
- User-provided ADS-B snapshot statistics from `aircraft.json`
  - source_status: user-provided session data
  - raw file itself was not attached to this KB operation
