# citizen-science-pipelines — TASKS.md

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

Backlog for **citizen-science-pipelines** (slug: `citizen-science-pipelines`), an open, reusable,
reproducible toolkit for cleaning, validating, labeling, and de-identifying citizen-science data.
See `PLAN.md` for full context.

## How these tasks map to Elyos

Each task below becomes an Elyos **Task JSON** validated against
`packages/schema/src/schemas.ts`. Field mapping:

- **id** — stable slug `csp-<area>-NNN` (e.g. `csp-engine-002`).
- **title** — the task title in the milestone table.
- **project** — `citizen-science-pipelines`.
- **type** — one of `code | research | writing | data | design-spec | maintenance`.
- **lane** — `donated` (default; no funded tasks planned. Any `funded` task must add
  `fundedBudgetUsd`).
- **priority** — `high | medium | low`.
- **domain** — array, e.g. `["software","citizen-science","open-data","biodiversity","privacy"]`.
- **riskTier** — `low | medium | high`. Privacy/sensitive-species, licensing, and partner-handoff
  tasks are `medium`; the sensitive-species generalization rule pack escalates to `high` for its
  expert sign-off; pure scaffolding/engine tasks are `low`.
- **urgent** — boolean (default `false`; no time-critical work in this project).
- **deliverable** — `pr | dataset | document | translation`.
- **tokenEstimate** — `small | medium | large` (maps to the Size column).
- **status** — `open | in-progress | review | delivered | done` (all start `open`).
- **context / objective / acceptanceCriteria[] / resources[] / output** — per task.
- **requestor** — partner/steward; `TO BE SECURED` where unknown.
- **verifiedNeed** — `true` only once a specific partner/need is confirmed; otherwise `false`.
- **outputLicense** — `MIT` for code; `CC-BY-4.0` for rule packs/specs/docs; processed-data outputs
  follow the source license (recorded per dataset).

Size legend: small ≈ tokenEstimate `small`, med ≈ `medium`, large ≈ `large`.

---

## Milestone M0 — Foundation & policy (cold-start)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| csp-repo-001 | Monorepo + pnpm workspaces + CI (build/test/lint) | code | small | low | pr | — | Maintainer |
| csp-engine-002 | `@csp/spec` schemas + `@csp/engine` deterministic runner + run manifest + double-run replay test | code | large | medium | pr | csp-repo-001 | Maintainer |
| csp-policy-003 | License/PII/sensitivity policy + non-destructive & reproducibility charter | writing | medium | medium | document | csp-repo-001 | License+PII reviewer |
| csp-partner-004 | Partner outreach + "secured" criteria + fallback dataset selection (runs in parallel) | research | small | medium | document | — | Steward |

**Acceptance criteria — key tasks**

- **csp-engine-002** (spec + deterministic engine)
  - `@csp/spec` defines + JSON-Schema-validates `PipelineSpec`, `ValidationRule`,
    `CleaningTransform`, `PrivacyRule`, `ChangeLedgerEntry`, `ValidationReport`, `RunManifest`.
  - Engine runs an ordered pipeline with a fixed seed and emits a complete W3C-PROV-style manifest
    (inputs + content hashes, steps + config hashes + tool versions, outputs + hashes).
  - **Double-run replay test:** running the same spec on the same inputs twice yields byte-identical
    output; CI fails on any diff.
  - Non-deterministic operations (unseeded RNG, wall-clock, locale-dependent sort) are statically
    forbidden / linted.
- **csp-policy-003** (governance charter)
  - Enumerates accepted source licenses (CC0/CC-BY/PD/open-gov; ODbL with share-alike; CC-BY-NC
    under the NC policy) and exclusions (no clear license; restricted-terms platforms; no scraping).
  - States the non-destructive, reproducibility, and **privacy fail-closed** invariants and the
    no-PII/sensitive-data-in-repo rule.
  - Defines the per-record license-tracking + "most-restrictive governs packaged output" rule.
  - Provides a reviewable license/PII/sensitivity gate checklist applied to every source.
- **csp-partner-004** (partner outreach — parallel from M0)
  - Records the objective "secured" criteria: signed DUA + named contact + passing acceptance test.
  - Identifies ≥ 1 candidate partner *category* and opens ≥ 1 outreach thread.
  - Selects an openly-licensed public fallback dataset (e.g. a CC0 GBIF subset) for the no-partner path.

**M0 Definition of Done:** monorepo + CI green; spec schemas + deterministic engine merged with the
double-run replay test passing; governance charter + gate checklist committed; partner-outreach
thread opened and fallback dataset identified. No real partner data processed yet.

---

## Milestone M1 — Validation engine + standards I/O

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| csp-validate-005 | `@csp/validate` rule engine (schema/range/enum/temporal/spatial/uniqueness/cross-field) + severities | code | large | low | pr | csp-engine-002 | Maintainer |
| csp-validate-006 | Taxonomic-name-resolution validator against pinned authoritative backbone | code | medium | medium | pr | csp-validate-005 | Domain reviewer |
| csp-adapter-007 | Darwin Core + Frictionless Data Package parse-in / pack-out adapter (canonical model) | code | medium | medium | pr | csp-engine-002 | Maintainer |
| csp-pipeline-018 | First reference pipeline on an openly-licensed sample (validate→report→export), reproducible | data | medium | medium | dataset | csp-validate-005, csp-adapter-007 | Domain reviewer |

**Acceptance criteria — key tasks**

- **csp-validate-005** (validation engine)
  - Pure, side-effect-free validators emit a `ValidationReport` (per-record + summary) with
    `error|warn|info` severities; **never auto-deletes** flagged records.
  - Covers schema/type, enum, numeric range, regex, temporal (parseable, not future, in window),
    spatial (coordinate-in-region, precision), uniqueness/duplicates, and cross-field consistency.
  - Report records a before/after error-rate pair for the success metric.
  - Golden-fixture tests (known-valid pass + deliberately malformed fail) pass in CI on synthetic data.
- **csp-validate-006** (taxonomic validation)
  - Resolves taxon names against a **pinned, versioned** authoritative backbone (e.g. GBIF backbone /
    Catalogue of Life); the version is recorded in the run manifest for reproducibility.
  - Flags unmatched/ambiguous/synonym names as `warn`/`error` with the matched concept; never
    silently rewrites a name without recording it in the change-ledger.
- **csp-adapter-007** (DwC + Frictionless I/O)
  - Parses Darwin Core (and DwC-Archive) and Frictionless Data Package into the canonical record
    model; packs outputs back as conformant DwC-Archive / Frictionless Data Package.
  - Output conformance validated in CI; license + provenance carried through to the packaged output.

**M1 Definition of Done:** validation engine + taxonomic check + DwC/Frictionless I/O merged with
golden-fixture + conformance tests green; one reference pipeline runs on an openly-licensed sample
end-to-end and **reproduces byte-identically**; before/after error-rate emitted.

---

## Milestone M2 — Cleaning (non-destructive) + privacy & sensitive-data module (hardened)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| csp-clean-008 | `@csp/clean` non-destructive transforms + change-ledger (normalize/dedupe/convert/parse) | code | large | medium | pr | csp-validate-005 | Domain reviewer |
| csp-privacy-009 | `@csp/privacy` PII detection/removal + k-anonymity + fail-closed invariant | code | large | medium | pr | csp-engine-002, csp-policy-003 | License+PII reviewer |
| csp-privacy-010 | Sensitive-species generalization rule pack (geo/date coarsening) + conservation-expert sign-off | data | medium | high | dataset | csp-privacy-009 | Conservation expert reviewer |
| csp-privacy-011 | PII/sensitivity detection methodology codified as a reviewable gate artifact | writing | small | medium | document | csp-privacy-009, csp-policy-003 | License+PII reviewer |

**Acceptance criteria — key tasks**

- **csp-clean-008** (non-destructive cleaning)
  - Every transform appends a `ChangeLedgerEntry` (`recordId, field, before, after, ruleId,
    reversible`); the input artifact's content hash is **unchanged** after a run (test-enforced).
  - Provides normalize/canonicalize, deduplicate, unit-convert, date-parse; **never imputes or
    fabricates** values; deduplication records which record was kept and why.
  - Deterministic: same input + spec ⇒ byte-identical cleaned output + ledger.
- **csp-privacy-009** (PII module, fail-closed)
  - Detects + removes/pseudonymizes living-individual PII (names, emails, usernames, precise home
    coords, device IDs); runs a quasi-identifier **k ≥ 5** check.
  - **Fail-closed invariant test:** no output contains a denylisted PII field, and no coordinate
    finer than its record's permitted precision; on detection uncertainty the pipeline coarsens to
    the coarsest configured level or halts.
- **csp-privacy-010** (sensitive-species generalization — **high-risk, expert-gated**)
  - Resolves taxa against a sensitivity source (IUCN status / national / partner list, partner
    overrides) and generalizes coordinates to an agreed grid/centroid + coarsens dates + suppresses
    exact-locality text.
  - Default fail-closed: unknown sensitivity ⇒ generalize. Generalization is never reversible from
    the output.
  - **Credentialed conservation/data-steward sign-off recorded** before the rule pack is marked
    usable; no sensitive-taxa dataset is processed for reliance until then.

**M2 Definition of Done:** non-destructive cleaning with change-ledger (originals provably
unchanged); PII module fail-closed with passing invariant tests; sensitive-species generalization
rule pack **signed off by a conservation expert**; detection methodology committed as a gate
artifact. The privacy surface is complete **before** any real platform adapter (M3) touches real data.

---

## Milestone M3 — Labeling/consensus + first real platform adapter

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| csp-label-012 | `@csp/label` assisted labeling + multi-annotator consensus + calibration + label provenance | code | large | medium | pr | csp-validate-005 | Domain reviewer |
| csp-adapter-013 | First real `@csp/adapter-*` against an openly-licensed platform/export (license+sensitivity vetted) | code | medium | medium | pr | csp-adapter-007, csp-privacy-010 | License+PII reviewer |
| csp-pipeline-014 | End-to-end pipeline (validate→clean→deidentify→label→export) on a real openly-licensed dataset, reproducible | data | medium | medium | dataset | csp-clean-008, csp-privacy-010, csp-label-012, csp-adapter-013 | Domain reviewer |

**Acceptance criteria — key tasks**

- **csp-adapter-013** (first real adapter)
  - Source license confirmed to permit reuse + derivatives (or licensed-in via partner agreement);
    **no scraping of restricted-terms platforms**; license snapshot + provenance recorded.
  - Sensitive taxa in the source are routed through `csp-privacy-010` generalization before any export.
  - ODbL/CC-BY/NC attributions and per-record licenses are preserved through the pipeline.
- **csp-pipeline-014** (full end-to-end, reproducible)
  - Runs the full step chain on a real openly-licensed dataset and **reproduces byte-identically**.
  - Output is standards-conformant (DwC/Frictionless), carries a complete provenance manifest, and
    passes the privacy/sensitivity invariant tests. Not yet partner-accepted.

**M3 Definition of Done:** labeling/consensus + one real adapter merged; full validate→clean→
deidentify→label→export pipeline runs reproducibly on a real openly-licensed dataset with privacy
and provenance gates green (not yet partner-delivered).

---

## Milestone M4 — Partner pilot + closed loop (the deed)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| csp-partner-015 | Secure partner project + data-use agreement + agreed rule packs / generalization defaults | research | medium | medium | document | csp-partner-004 | Steward |
| csp-handoff-016 | Toolkit used on partner's real dataset; cleaned/validated/de-identified output accepted and used | data | medium | medium | dataset | csp-partner-015, csp-pipeline-014 | Steward + License+PII reviewer + Conservation expert reviewer |

**Acceptance criteria — key tasks**

- **csp-partner-015** (secure partner — runs from M0 via `csp-partner-004`)
  - "Secured" requires **all three**: signed **DUA**, a **named accountable contact**, and a passing
    **acceptance test** on a sample cleaned output.
  - Generalization defaults (grid resolution, date granularity, suppression rules), sensitivity
    list, and taxonomic backbone are agreed and recorded jointly.
  - **Fallback** if no partner: run on a fully openly-licensed public dataset + publish output +
    manifest + datasheet + commission an independent expert spot-check (does **not** meet
    Definition of Shipped).
- **csp-handoff-016** (closed loop — the deed)
  - The cleaned/validated/de-identified output is **formally accepted and used** by the partner.
  - Reproducibility independently verified; provenance published; sensitive-handling expert sign-off
    recorded where sensitive taxa present; before/after error reduction recorded.
  - Zero PII / finer-than-permitted sensitive coordinates in the delivered output.

**M4 Definition of Done:** project-level **Definition of Shipped** met — the open-source toolkit has
been used by a real citizen-science project on a real dataset, the output is accepted and used,
the run is reproducible, provenance is published, and the privacy/sensitivity gates passed with
expert sign-off where applicable.

---

## Milestone M5 — Sustain & scale (post-delivery)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
|---|---|---|---|---|---|---|---|
| csp-ops-017 | Ops/contribution runbook + rule-pack governance + outcome dashboard + maintenance rotation | maintenance | medium | low | document | csp-handoff-016 | Maintainer |

**Acceptance criteria — csp-ops-017**
- Runbook covers running a pipeline, adding an adapter, adding/versioning a rule pack, and partner
  handoff; rule-pack governance requires re-sign-off for any sensitive-species generalization change.
- Dashboard tracks pilots delivered/used, datasets cleaned, before/after error reduction,
  reproducibility-replay pass rate, and privacy-gate status (not stars/downloads).
- Named maintenance rotation in place.

**M5 Definition of Done:** project is sustainably maintained with outcomes tracked, rule-pack
governance documented, and a rotation owning it after the first delivery.

---

## Backlog / future

| ID | Title | Type | Size | Risk | Deliverable | Notes |
|---|---|---|---|---|---|---|
| csp-adapter-019 | Second platform adapter (different domain than first) | code | large | medium | pr | Proves pluggability across the fragmented landscape |
| csp-exec-020 | Python executor for the language-neutral pipeline spec | code | large | medium | pr | Meets the Python/R ecosystem where it lives; spec already neutral |
| csp-consensus-021 | Upgrade labeling consensus to Dawid–Skene / EM estimator | code | large | medium | pr | When annotation volume supports it; sparsity-aware |
| csp-croissant-022 | Croissant ML metadata + Datasheet emitter for cleaned outputs | code | medium | low | pr | Documentation-completeness metric; reuse with open-data-datasheets |
| csp-i18n-023 | Internationalized rule-pack messages + docs | writing | medium | low | translation | Lower barrier for non-English-speaking projects |
| csp-edu-024 | "How this data was cleaned" reproducibility explainer for the public | writing | small | low | document | Open-science literacy; secondary benefit |

> **Fan-out note (csp-i18n-023):** the target language set is intentionally *not* enumerated — it
> depends on partner/community demand, which is `TO BE SECURED`. The task JSON is therefore a single
> representative task; concrete per-language items expand once a language set is confirmed.
> Translations stay source-compatible and never relicense copyrighted source.

---

## Example task JSON

Complete, schema-valid Task JSON for the first M0 task (`csp-repo-001`):

```json
{
  "id": "csp-repo-001",
  "title": "Monorepo + pnpm workspaces + CI (build/test/lint)",
  "project": "citizen-science-pipelines",
  "type": "code",
  "lane": "donated",
  "priority": "high",
  "domain": ["software", "citizen-science", "open-data", "reproducibility"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "pr",
  "tokenEstimate": "small",
  "status": "open",
  "context": "citizen-science-pipelines is an open, reusable, reproducible toolkit for cleaning, validating, labeling, and de-identifying citizen-science data. This is the cold-start foundation task: there is no repo scaffold yet. The codebase must follow Elyos conventions — TypeScript, ESM, pnpm workspaces — with a strict separation between the agent-neutral core (@csp/spec, @csp/engine, @csp/validate, @csp/clean, @csp/privacy, @csp/label, @csp/cli) and platform-specific adapters (@csp/adapter-*). No real partner data is processed; no partner is yet secured.",
  "objective": "Scaffold the pnpm-workspace monorepo with placeholder packages (@csp/spec, @csp/engine, @csp/validate, @csp/clean, @csp/privacy, @csp/label, @csp/cli, and an example @csp/adapter-* stub) and a CI pipeline running build, test, and lint, so all later tasks have a green, reproducible baseline.",
  "acceptanceCriteria": [
    "pnpm workspace with packages: @csp/spec, @csp/engine, @csp/validate, @csp/clean, @csp/privacy, @csp/label, @csp/cli, and an example @csp/adapter-* stub",
    "TypeScript + ESM configured consistently across packages with a shared tsconfig base",
    "pnpm build && pnpm test && pnpm lint all pass on a clean checkout",
    "CI workflow runs build/test/lint on push and PR and is green",
    "MIT LICENSE for code; README states the reproducibility, non-destructive, and privacy-fail-closed constraints and the open-license-only data policy; DCO sign-off configured",
    "A CI check rejecting any committed coordinate/PII-shaped fixtures outside the synthetic allowlist is stubbed in"
  ],
  "resources": [
    "planning/projects/citizen-science-pipelines/PLAN.md",
    "planning/projects/citizen-science-pipelines/TASKS.md",
    "CLAUDE.md",
    "docs/good-deed-definition.md",
    "packages/schema/src/schemas.ts"
  ],
  "output": "A PR adding the citizen-science-pipelines monorepo scaffold (pnpm workspaces, package stubs, shared TS/ESM config, lint setup, MIT license, README stating the core constraints) and a green CI workflow for build/test/lint.",
  "requestor": "TO BE SECURED",
  "verifiedNeed": false,
  "outputLicense": "MIT"
}
```

---

## Generated task index

Every backlog row above (M0–M5 plus the future backlog) now has a corresponding schema-valid
`tasks/<id>.json`, validated against `packages/schema/src/schemas.ts`. The full set:

| ID | Milestone | Type | Deliverable | Risk |
|---|---|---|---|---|
| csp-repo-001 | M0 | code | pr | low |
| csp-engine-002 | M0 | code | pr | medium |
| csp-policy-003 | M0 | writing | document | medium |
| csp-partner-004 | M0 | research | document | medium |
| csp-validate-005 | M1 | code | pr | low |
| csp-validate-006 | M1 | code | pr | medium |
| csp-adapter-007 | M1 | code | pr | medium |
| csp-pipeline-018 | M1 | data | dataset | medium |
| csp-clean-008 | M2 | code | pr | medium |
| csp-privacy-009 | M2 | code | pr | medium |
| csp-privacy-010 | M2 | data | dataset | **high** |
| csp-privacy-011 | M2 | writing | document | medium |
| csp-label-012 | M3 | code | pr | medium |
| csp-adapter-013 | M3 | code | pr | medium |
| csp-pipeline-014 | M3 | data | dataset | medium |
| csp-partner-015 | M4 | research | document | medium |
| csp-handoff-016 | M4 | data | dataset | medium |
| csp-ops-017 | M5 | maintenance | document | low |
| csp-adapter-019 | backlog | code | pr | medium |
| csp-exec-020 | backlog | code | pr | medium |
| csp-consensus-021 | backlog | code | pr | medium |
| csp-croissant-022 | backlog | code | pr | low |
| csp-i18n-023 | backlog | writing | translation | low |
| csp-edu-024 | backlog | document | document | low |

Notes:
- **All tasks carry `verifiedNeed: false` and `requestor: "TO BE SECURED"`** — no partner is yet
  secured (per `PLAN.md`); the flag flips only once `csp-partner-015` meets all three secured criteria.
- **`csp-privacy-010` is the one `high`-risk task** — sensitive-species generalization requires
  credentialed conservation/data-steward expert sign-off before any reliance (preserved in its
  `context`/`acceptanceCriteria`).
- **No fabricated fan-out:** partner, dataset, sensitivity list, taxonomic backbone, and i18n
  language set are all `TO BE SECURED`/open-ended, so each row is emitted as a single representative
  task rather than invented per-item JSONs.
- Output licenses: code `MIT`; specs/rule-packs/docs `CC-BY-4.0`; processed-data outputs follow the
  source license (recorded per run); translations stay source-compatible.
