# HPR KB Schema

## Knowledge Stream

Each durable topic is stored in `kb/streams/<stable-slug>.md` and uses these sections where relevant:

1. Intent
2. Current Canonical State
3. Confirmed Facts
4. Requirements
5. Constraints
6. Decisions
7. Architecture / Model
8. Business Rules / Invariants
9. Important Entities
10. Important Numbers / Parameters
11. Rejected / Superseded Alternatives
12. Open Questions
13. Pending Actions
14. Related Streams
15. Evolution
16. Sources

## Decision status

- `CANONICAL` — current committed direction.
- `TENTATIVE` — supported working direction but not fully committed.
- `DEFERRED` — deliberately postponed.

## Stream status

- `ACTIVE`
- `STABLE`
- `EXPLORATORY`
- `PAUSED`

## Maturity

- `LOW`
- `MEDIUM`
- `HIGH`

## Provenance

Authoritative raw transcripts, when available, belong under `raw/chatgpt/YYYY/MM/` and are immutable evidence. If no authoritative transcript is available, stream sources must explicitly record `raw_available: false` and `source_status: context-derived`.

## Canonicalization rules

Later explicit decisions supersede earlier proposals when they materially conflict. Implemented state outranks proposals. Brainstorming must not be promoted into canonical state. Significant superseded architecture is preserved under Evolution or Rejected / Superseded Alternatives rather than presented as current truth.
