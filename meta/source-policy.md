# HPR KB Source Policy

## Purpose

Canonical HPR knowledge must distinguish authoritative standards and regulator material from vendor, project, community and secondary sources. Source quality is part of the knowledge contract, not an editorial afterthought.

## Authority tiers

### Tier 1 — Standards bodies, regulators, public authorities

Preferred for protocol semantics, regulatory requirements, spectrum use, safety systems and official operational definitions.

Examples: ICAO, ITU-R, IMO, FAA, EASA, EUROCONTROL, WMO, NOAA/NWS, Cospas-Sarsat and national maritime/aviation authorities.

Rules:

- Use the current in-force recommendation or regulation when one exists.
- If an older edition is used because it exposes detail that is easier to quote or search, explicitly mark it as superseded and also cite the current edition.
- Regulatory and safety claims must not rely only on community documentation.

### Tier 2 — Protocol owner / specification owner

Authoritative for the owner's protocol definition, implementation contract and licensing terms when no public SDO standard is the primary specification.

Examples: FLARM for FAMP.

Rules:

- Use owner documentation for packet semantics and license conditions.
- Do not treat owner marketing claims as independent evidence for comparative performance.

### Tier 3 — Maintainer / project documentation

Useful for implementation-specific facts about an open project or ecosystem.

Examples: Open Glider Network documentation, TAPR APRS specification.

Rules:

- Suitable for what the project supports or how its implementation behaves.
- Not sufficient by itself for regulatory, legal or safety claims.

### Tier 4 — Secondary technical sources

Blogs, forums, community reverse engineering, aggregators and informal explainers.

Rules:

- May be used for discovery, troubleshooting or cross-checking.
- Must not be the sole source for a canonical technical fact when a Tier 1–3 source exists.

## Citation rules

1. Every non-trivial protocol, regulatory, licensing or safety claim in a canonical KB stream must have a traceable source.
2. Blog-candidate material must have either inline citations or a section-by-section source register before publication.
3. Source version and status matter. Prefer current/in-force standards and record revision dates when material.
4. A protocol's existence, its payload semantics, and permission to receive/redistribute it are three different claims and may require three different sources.
5. Legal or licensing uncertainty must be written as uncertainty. Do not infer redistribution rights merely because a signal is receivable over the air.
6. HPR implementation decisions may cite HPR repositories and tests, but those do not replace external standards for external protocol semantics.

## Evidence labels

Use these labels in source registers where useful:

- `PRIMARY-STANDARD` — current SDO recommendation/specification.
- `PRIMARY-REGULATOR` — regulator or public authority.
- `PROTOCOL-OWNER` — owner-maintained protocol specification or license.
- `PROJECT-AUTHORITY` — authoritative documentation for a specific open project.
- `HPR-IMPLEMENTATION` — HPR repository, test or measured result.
- `SECONDARY` — supporting explanation only.

## Publication gate

A blog-candidate stream is publishable only when:

- each technical section is covered by at least one Tier 1 or Tier 2 source where such a source exists;
- project-specific ecosystem claims identify the Tier 3 project source;
- legal/licensing statements cite the applicable authority or license text;
- no superseded standard is presented as current;
- claims not meeting the above are explicitly marked `UNVERIFIED` or removed.
