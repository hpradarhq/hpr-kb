# Front-End Quality Standard

**Status:** ACTIVE  
**Maturity:** HIGH  
**Last updated:** 2026-08-30

## Intent

Establish one reusable front-end design and release standard for the HPRadar product family. The standard converts external front-end guidance into project-specific, testable quality gates.

## Current Canonical State

- Front-End Checklist is the primary external rule corpus for HPR front-end implementation reviews.
- HPR projects do not copy the upstream corpus. They reference a pinned upstream commit and maintain a smaller HPR applicability profile.
- Quality checks begin during design and component implementation, not only before release.
- Critical and High applicable findings block release. Medium findings require resolution or a documented exception. Low findings are backlog candidates.
- Automated checks are preferred for objective assertions. Visual review remains necessary for product direction and map-specific interaction quality.
- First adopter: `hpradarhq/hpr-traffic-api`.

## Requirements

Every production front end must cover the applicable rules in these domains:

1. HTML structure and metadata.
2. CSS, responsive layout and browser compatibility.
3. JavaScript correctness and progressive degradation.
4. Accessibility: semantics, names, keyboard, focus, contrast, reduced motion and touch targets.
5. Performance: loading strategy, asset size, rendering stability and runtime errors.
6. Security and privacy: HTTPS, CSP, trusted dependencies, SRI or self-hosting, safe external links and data minimization.
7. Images and media: dimensions, responsive delivery, alternative text and lazy loading where appropriate.
8. Testing across supported viewports and critical user journeys.
9. SEO and internationalization only where the surface is public/indexable or multilingual.

## HPR Design Rules

- Define user journeys, information hierarchy, responsive behavior, empty/loading/error states and keyboard behavior before visual polish.
- Map applications preserve usable map controls and one active overlay surface at a time.
- Mobile is a distinct interaction layout, not a compressed desktop layout.
- Primary interactive targets are at least 44 × 44 CSS px on touch layouts.
- Every icon-only control has an accessible name.
- Focus order follows visual and task order; focus remains visible.
- Status is not communicated by color alone.
- Motion respects `prefers-reduced-motion`.
- External production dependencies are pinned and either self-hosted or protected with appropriate integrity and cross-origin metadata.
- Runtime JSX compilers and development CDN frameworks are prototype-only.
- Production pages declare doctype, UTF-8, language and responsive viewport metadata.

## Review Workflow

1. Classify the changed surface: public page, authenticated app, map/dashboard, form, media-heavy or prototype.
2. Select applicable upstream categories and record non-applicable categories.
3. Check Critical and High rules first.
4. Implement automated assertions for stable requirements.
5. Exercise critical journeys with keyboard and the supported viewport matrix.
6. Record exceptions with rule, reason, owner and expiry/review date.
7. Attach the audit result to the pull request or release evidence.

## First Adoption: HPR Traffic API

Repository: `hpradarhq/hpr-traffic-api`

The existing V5 gates already cover responsive layouts, touch targets, accessible labels, focus, contrast, browser journeys and production smoke tests. Adoption adds:

- this upstream corpus as the named source standard;
- a project applicability profile;
- dependency, metadata, security, reduced-motion and semantic checks;
- an FE-specific pull-request declaration;
- explicit prototype exclusions for `sample/template.html`.

## Constraints

- The upstream corpus changes over time. HPR decisions must cite a commit, not only the moving `main` branch.
- Not every rule applies to a real-time authenticated map application. SEO and PWA rules require explicit applicability decisions.
- Passing a generic checklist does not establish domain correctness, map usability or data correctness.
- The upstream repository did not expose a root `LICENSE` file at the pinned revision. HPR stores derived policy and links, not a wholesale copy of the corpus.

## Decisions

- **CANONICAL:** Use Front-End Checklist as the external FE review corpus.
- **CANONICAL:** Maintain HPR-specific executable gates in each product repository.
- **CANONICAL:** Do not vendor the complete upstream README/rule corpus into `hpr-kb`.
- **CANONICAL:** Start adoption with `hpradarhq/hpr-traffic-api`, then reuse the pattern across HPRadar FE repositories.

## Pending Actions

- Run and record the first full audit of the V5 production surface.
- Convert confirmed Critical/High gaps into executable CI assertions.
- Reuse the standard in HPR Atlas and the shared HPR UI template after the first adopter stabilizes it.

## Related Streams

- HPR Atlas
- HPRadarHQ

## Sources

- Front-End Checklist repository: https://github.com/thedaviddias/Front-End-Checklist
- Pinned source commit: `30756a79b2f7d4363ac592710146c8e28fa9f1b5` (2026-08-14)
- Upstream rule browser: https://frontendchecklist.io/rules
- First adopter: https://github.com/hpradarhq/hpr-traffic-api
- User directive, 2026-08-30; `raw_available: false`, `source_status: session-derived`.
