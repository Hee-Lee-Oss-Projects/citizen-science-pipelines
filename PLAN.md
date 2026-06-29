# PLAN — citizen-science-pipelines

> Status: Draft · Version: 0.1.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

## Executive summary

Citizen-science projects — biodiversity observation networks, community air-quality and
water-quality monitoring, phenology and weather logging, transcription and astronomy
classification efforts — collectively generate an enormous, scientifically valuable stream of
volunteer-contributed observations. That stream is also **messy by construction**: inconsistent
formats, typos, unit confusion, duplicates, taxonomic name drift, implausible coordinates and
dates, and — critically — inadvertent **personal data** (observer names, home coordinates) and
**conservation-sensitive locations** (precise positions of poachable or collectible species).
Most projects clean and validate this data with one-off, undocumented, non-reproducible scripts,
or not at all. The work is repeated independently by thousands of projects and is rarely shared.

This project builds an **open, reusable, reproducible toolkit for cleaning, validating, labeling,
and de-identifying citizen-science data** — plus a small library of declarative, shareable
pipeline specs and validation rule packs aligned to community standards (Darwin Core, Frictionless
Data, W3C PROV). The deliverable is **tooling and method, not a dataset**: we do not host, sell,
or claim ownership of anyone's observations. We give projects a transparent, auditable way to turn
raw contributions into trustworthy, well-documented, privacy-respecting data — and we give the
public a reproducible record of exactly what was done to the data.

Three constraints define the project and are non-negotiable: **(1) reproducibility** — every
pipeline run is deterministic and produces a complete provenance manifest, so any third party can
re-run it and get byte-identical output; **(2) non-destructive transformation** — original data is
never silently mutated; every change is recorded in an auditable change-ledger; **(3) privacy &
do-no-harm, fail-closed** — no living-individual PII and no fine-grained sensitive-species
coordinates ever leave a pipeline, and when detection is uncertain the pipeline generalizes harder
or halts rather than leaking.

Risk tier is **medium**, with one surface treated to a **high-stakes bar**: handling of
conservation-sensitive species locations can cause real-world harm (poaching, collection,
disturbance) if done wrong, and therefore requires **credentialed conservation/data-steward
sign-off** before any project relies on it. This plan is honest about its central gap: **no partner
citizen-science project is yet secured.** Until a real project adopts the toolkit on a real dataset
and accepts the cleaned/validated output, this is excellent infrastructure, not a delivered good
deed. Securing that first pilot is the critical-path dependency, marked `TO BE SECURED` throughout.

## Problem & beneficiaries

**Who is helped.**
- **Citizen-science projects and their coordinators** — small conservation NGOs, university labs,
  museum/herbarium data teams, community monitoring groups, and volunteer-run platforms — who need
  to clean and validate volunteer data but lack the engineering capacity to do it reproducibly.
- **The volunteer contributors** themselves — their effort becomes scientifically usable, and
  their privacy (and the safety of the species they report) is actively protected.
- **Downstream researchers and data aggregators** (e.g. those consuming Darwin Core archives) —
  who receive better-documented, validated, provenance-carrying data.
- **The public** — a transparent, open, replicable standard for how community science data is
  cleaned, which is itself a contribution to scientific integrity and reproducibility.

**The verified need (general vs. specific).** The *general* need is well established: data quality
and reproducibility are repeatedly cited as the primary barriers to scientific use of
citizen-science data, and the field has produced explicit data-quality frameworks (e.g. GBIF data
quality flags, Darwin Core conformance) precisely because raw contributions are unreliable. We
treat the general need as real. The **specific, per-partner need is `TO BE SECURED`**: we have not
yet confirmed a named project that has agreed to adopt the toolkit and accept its output. Until a
specific project confirms this, individual tasks carry `verifiedNeed: false`. This honesty matters
because "delivered, not merged" requires the output to be *used by a beneficiary*, not just built.

**Partner org.** `TO BE SECURED`. Candidate *categories* (not commitments): biodiversity/natural
history platforms and aggregators publishing under open licenses, community environmental
monitoring networks, museum/herbarium digitization programs, and academic citizen-science labs.
No partner relationship may be claimed as real until a Memorandum of Understanding / data-use
agreement and an accepted, used output exist (see Milestone M4).

**Who is NOT a beneficiary.** No for-profit entity is the primary beneficiary. If a commercial data
broker wants the tooling, they get it under the same open license as everyone else — no privileged
access, no bespoke features. We do not build features whose primary purpose is to make someone's
proprietary dataset more saleable.

## Goals and non-goals

**Goals**
- Ship an open, agent-neutral toolkit (`@csp/*`) for validating, cleaning, labeling, and
  de-identifying citizen-science data, driven by **declarative, shareable pipeline specs**.
- Make every pipeline run **reproducible**: deterministic, content-addressed inputs, pinned tool
  versions, fixed seeds, and a complete W3C-PROV-style run manifest that a third party can replay.
- Make every transformation **non-destructive and auditable**: originals preserved, every change
  recorded in a change-ledger with the rule that caused it.
- Provide a **privacy & sensitive-data module** that detects observer PII and conservation-sensitive
  locations and **generalizes or removes** them, fail-closed, with conservation-expert-approved
  generalization rules.
- Align outputs to community standards (Darwin Core, Darwin Core Archive, Frictionless Data
  Package, Croissant metadata, W3C PROV) so results drop into existing ecosystems.
- Provide reusable, versioned **validation rule packs** (taxonomic, spatial, temporal, range,
  cross-field) and an **assisted-labeling / consensus** workflow for human verification.
- Prove value through real pilots: a real project adopts the toolkit and *uses* the output.

**Non-goals**
- We do **not** host, mirror, sell, or claim ownership of anyone's observation data.
- We do **not** scrape proprietary or restricted-terms databases (e.g. eBird raw downloads have
  restrictive terms; any such source is excluded unless the partner brings an explicit agreement).
- We do **not** invent, synthesize, or impute observations to "fill gaps" in scientific data;
  cleaning corrects format/encoding/duplication, it does not fabricate records.
- We do **not** make ecological or scientific *conclusions* (no species-distribution modeling,
  rankings, or interpretation) — we produce cleaned, validated, documented data and the tools to do so.
- We do **not** publish or expose fine-grained sensitive-species coordinates, ever.
- We do **not** build civic/governance/election-monitoring citizen-science features that would
  require partisan judgement; if such a use arises it is out of scope unless strictly neutral and
  separately governed (see Data, licensing & compliance).
- We do **not** auto-publish cleaned data anywhere; a human at the partner reviews and publishes.
- We are **not** a hosted SaaS in v1 — the toolkit runs locally / in the partner's own CI.

## Success metrics (outcomes)

Outcome-centric and beneficiary-facing. Vanity metrics (lines of code, tools written, GitHub stars)
are explicitly excluded. Baselines are zero (greenfield); targets are first-year aspirations to be
re-baselined after the first pilot.

| Outcome | Metric | Baseline | Target (12 mo) |
|---|---|---|---|
| Real-world adoption (the headline) | Citizen-science projects that **adopt the toolkit on a real dataset and accept/use the output** | 0 | ≥ 2 pilots delivered and used |
| Loop is closed | Partners with a signed data-use agreement + ≥ 1 accepted, *used* cleaned output | 0 | ≥ 1 |
| Reproducibility (hard invariant) | Delivered pipelines that re-run to **byte-identical output** on independent replay | n/a | **100%** |
| Privacy safety (hard invariant) | Living-individual PII or finer-than-allowed sensitive-species coordinates present in any delivered output | 0 | **0** |
| Data-quality improvement | Validation-error / duplicate rate reduced, measured before vs. after, per delivered dataset | recorded at intake | ≥ 50% reduction in flagged errors (per dataset, vs. its own intake baseline) |
| Expert assurance | Sensitive-species generalization rule packs with **conservation-expert sign-off** before reliance | 0 | 100% of sensitive-handling features signed off before any partner relies on them |
| Reuse | Pipeline specs / rule packs reused across ≥ 2 distinct projects or domains | 0 | ≥ 1 spec or rule pack reused externally (verifiable) |
| Openness | Delivered pipelines emitting a complete provenance manifest + standards-conformant output | n/a | 100% (DwC/Frictionless conformance validated in CI) |
| Documentation completeness | Delivered datasets carrying a Datasheet/metadata record describing cleaning applied | n/a | 100% |

Notes on attribution: "adopted and used" must be externally verifiable — a partner's written
confirmation, a published Darwin Core archive citing the pipeline, or a merged change in the
partner's own repository. Self-reported reuse does not count. "Error reduction" is measured against
each dataset's own recorded intake baseline, never across datasets (they are not comparable).

## Scope

**In scope**
- **Pipeline spec & engine:** a declarative, versioned, language-neutral pipeline format (steps:
  validate / clean / label / de-identify / export) and a deterministic reference executor.
- **Validation:** schema/type/enum/range/regex, taxonomic-name resolution checks, spatial
  plausibility (coordinate-in-country, land/sea, precision), temporal plausibility (date parseable,
  not future, within project window), uniqueness/duplicate detection, cross-field consistency.
- **Cleaning (non-destructive):** normalization, canonicalization, unit conversion, date parsing,
  deduplication — all logged to a reversible change-ledger; originals preserved.
- **Privacy & sensitive-data module:** observer-PII detection and removal; conservation-sensitive
  location detection and **generalization** (coordinate coarsening to grid / centroid, date
  coarsening, name suppression), fail-closed.
- **Labeling & consensus:** assisted human verification, multi-annotator consensus, calibration on
  known-answer items, label provenance.
- **Standards I/O & adapters:** Darwin Core / DwC-Archive, Frictionless Data Package, Croissant
  metadata, W3C PROV manifests; per-platform format **adapters** kept out of the core.
- **Reproducibility infrastructure:** content hashing, tool-version pinning, run manifests,
  replay verification.
- **Rule packs & reference pipelines:** versioned, documented, reusable.

**Out of scope (will NOT do)**
- Hosting, mirroring, selling, or owning observation data.
- Scraping proprietary/restricted-terms databases; ingesting any source whose license forbids
  reuse/derivatives.
- Fabricating, imputing, or synthesizing scientific observations.
- Scientific interpretation: distribution modeling, rankings, trend conclusions, "is this species
  declining" answers.
- Publishing or exposing fine-grained sensitive-species coordinates.
- Partisan civic/governance/election-integrity data work.
- Automated, unattended publishing to any platform.
- A hosted multi-tenant SaaS (v1 is a local/CI toolkit).
- Processing controlled-access or human-subjects data requiring IRB / ethics-board approval (e.g.
  health citizen-science with patient data) — out of scope unless a partner brings the approvals.

## Solution approach & architecture

This is a **software toolkit with a strong data-governance core**, mirroring Elyos's
agent-neutral-core discipline: vendor/platform specifics live only in `adapters/`.

### High-level shape

```
+----------------------------+      +-------------------------------+
|  @csp/spec (protocol)      |      |  @csp/engine (reference exec) |
|  - PipelineSpec schema     |<---->|  - deterministic step runner  |
|  - ValidationRule          | uses |  - content hashing / seeds    |
|  - CleaningTransform       |      |  - run manifest (W3C PROV)    |
|  - PrivacyRule             |      |  - replay / reproduce verify  |
|  - ProvenanceRecord        |      +---------------+---------------+
+-------------+--------------+                      |
              |                       +-------------v-------------+
   +----------+-----------+           |  @csp/validate            |
   | @csp/clean           |           |  schema/range/taxonomic/  |
   | normalize/dedupe/    |           |  spatial/temporal/dupes   |
   | convert (ledgered)   |           +---------------------------+
   +----------+-----------+
              |               +-------------------------------+
   +----------v-----------+   |  @csp/privacy (fail-closed)   |
   | @csp/label           |   |  PII detect + remove          |
   | assisted + consensus |   |  sensitive-loc generalize     |
   +----------------------+   +---------------+---------------+
                                              |
                              +---------------v---------------+
                              |  @csp/adapter-<platform>      |
                              |  DwC / iNat-export / GBIF /   |
                              |  OSM(ODbL) / Frictionless     |
                              |  - parse in / pack out        |
                              |  - carry license + provenance |
                              +-------------------------------+
                       CLI: @csp/cli  (run | validate | reproduce | report)
```

### Components & tech stack

- **`@csp/spec`** — shared TypeScript types + JSON Schemas (ajv) for `PipelineSpec`,
  `ValidationRule`, `CleaningTransform`, `PrivacyRule`, `ChangeLedgerEntry`, `ValidationReport`,
  `ProvenanceRecord`/`RunManifest`. The contract between every other package. No platform logic.
- **`@csp/engine`** — the deterministic reference executor: resolves inputs by content hash, runs
  steps in order with fixed seeds, forbids non-deterministic operations, and emits a complete run
  manifest. Provides `reproduce` (re-run a manifest and assert byte-identical output).
- **`@csp/validate`** — pure, side-effect-free validators producing a `ValidationReport`
  (per-record + summary, with severities). Importable standalone.
- **`@csp/clean`** — non-destructive transforms; every change appends a `ChangeLedgerEntry`
  (original value, new value, rule id, reversible flag). Never mutates inputs in place.
- **`@csp/privacy`** — PII detection/removal + sensitive-location generalization, **fail-closed**.
  Generalization rules are data (grid resolution, sensitivity source) and are the high-stakes
  surface requiring expert sign-off.
- **`@csp/label`** — assisted labeling + multi-annotator consensus with calibration items and
  label provenance.
- **`@csp/adapter-*`** — one adapter per platform/format; parses source into the canonical record
  model and packs output (DwC-Archive, Frictionless Data Package) with license + provenance.
- **`@csp/cli`** — thin CLI: `run`, `validate`, `reproduce`, `report`. Runs locally or in the
  partner's CI. No network calls except explicit, declared input fetches.
- **Tooling/deploy** — TypeScript, ESM, pnpm workspaces (Elyos conventions); minimal, permissively
  licensed deps; no runtime service in v1. **Language note (key decision/open question):** much of
  the citizen-science ecosystem is Python (pandas/R). We keep the *pipeline spec language-neutral*
  (declarative JSON/YAML) so the spec is the durable artifact and a Python executor could be added
  later; the reference executor is TypeScript per Elyos conventions. We provide standards-conformant
  outputs (DwC, Frictionless) precisely so Python/R users consume results natively.

### Canonical data model & key records

- **`PipelineSpec`** (the durable, shareable artifact): `id`, `version`, `inputs[] {ref,
  contentHash, license, provenance}`, `steps[] {kind: validate|clean|label|deidentify|export,
  ruleRefs[], config, deterministic:true}`, `seed`, `toolVersions{}`, `outputs[] {format,
  licensePolicy}`. Specs are versioned and reusable across datasets.
- **`ValidationRule`**: `id`, `field`, `check` (type/range/enum/regex/taxonomic/spatial/temporal/
  uniqueness/crossField), `severity` (error|warn|info), `dwcTerm?`, `rationale`, `version`.
- **`ValidationReport`**: per-record findings + summary counts by rule/severity + `contentHash`;
  carries a before/after error-rate pair for the success metric.
- **`CleaningTransform`** + **`ChangeLedgerEntry`**: every cleaning op records
  `{recordId, field, before, after, ruleId, reversible}`; the ledger is the audit trail and makes
  cleaning explainable and (where flagged) reversible. **Originals are never edited in place.**
- **`PrivacyRule`**: `id`, `detect` (PII pattern / sensitivity-list lookup), `action`
  (suppress|generalize), `generalization {geoGridKm | dateGranularity | nameRedaction}`,
  `sensitivitySource` (e.g. IUCN status, national sensitive-species list, partner list),
  `failClosed:true`.
- **`ProvenanceRecord` / `RunManifest`** (W3C PROV-aligned): inputs (with content hashes),
  ordered steps (config hash + tool versions + seed), outputs (content hashes), operator, timestamp.
  Travels with every output and is what makes a run **reproducible** and a result **trustworthy**.

### Core invariants (enforced by tests, not just prose)

1. **Determinism / reproducibility:** same inputs + same spec ⇒ byte-identical output. CI runs each
   reference pipeline twice and fails the build on any diff. Non-deterministic operations (unseeded
   RNG, wall-clock, unordered map iteration, locale-dependent sorts) are statically forbidden.
2. **Non-destructive:** no transform mutates an input artifact in place; every change is in the
   change-ledger; an input's content hash is unchanged after a run.
3. **Privacy fail-closed:** if PII/sensitivity detection is uncertain or a sensitivity list is
   unavailable, the pipeline **generalizes to the coarsest configured level or halts** — it never
   emits finer precision on doubt. An invariant test asserts no output contains a coordinate finer
   than its record's permitted precision and no field on the PII denylist.
4. **Provenance-complete:** no `export` step may emit an output lacking a full run manifest +
   per-output license; a test fails the build otherwise.

### Key decisions

- **Spec-first, language-neutral.** The pipeline spec — not the executor — is the durable, shareable
  good. This is what lets one project's cleaning method be reused/audited by another.
- **Declarative rules as data.** Validation/cleaning/privacy rules are versioned data, recorded in
  every run, so behavior is auditable and reproducible rather than buried in code.
- **Adapters at the edge.** All platform/format specifics live in `@csp/adapter-*`, keeping the core
  agent-neutral and the toolkit portable across the fragmented citizen-science landscape.
- **Privacy is a first-class module, not a filter step.** Sensitive-location handling is the
  highest-harm surface and is designed/tested/expert-reviewed as such.
- **Standards-out.** Outputs conform to Darwin Core / Frictionless so they integrate with GBIF and
  the wider ecosystem without us building a parallel universe.

## Data, licensing & compliance

**This is the critical section** and is intentionally conservative. We process *other people's*
community-contributed data, so licensing and privacy are front-loaded as hard gates.

**Source licenses we accept (must permit reuse AND derivatives):**
- **CC0 1.0** — accepted (e.g. many GBIF-mediated records); provenance still recorded.
- **CC-BY 4.0** — accepted; attribution preserved through the pipeline and into outputs.
- **Public domain / open government** sources — accepted; provenance recorded.
- **ODbL (OpenStreetMap)** — accepted **with share-alike honored**: any derivative database/output
  incorporating OSM-derived data is released under ODbL and attributed "© OpenStreetMap
  contributors"; OSM-derived layers are kept separable so share-alike obligations are explicit.
- **CC-BY-NC** (common for some iNaturalist observations) — accepted **only** under the project's
  written NC policy: our toolkit and rule packs are non-commercial public-commons artifacts, and NC
  source data may be processed for a non-commercial partner, but NC status is recorded per record
  and **never relicensed**; mixed-license datasets keep per-record license and the most restrictive
  governs the packaged output. Default on any ambiguity: **exclude/escalate**, never default-allow.

**Excluded / never ingested:**
- Sources with **no clear license**, "all rights reserved," or terms forbidding reuse/derivatives.
- **Restricted-terms platforms** (e.g. eBird raw-download terms, proprietary databases) — **no
  scraping**, ever. Such data enters *only* if the partner holds and supplies an explicit agreement,
  recorded as provenance.
- Controlled-access, IRB-governed, or human-subjects data without the partner's approvals.

**Privacy / PII stance (no living-individual PII; aggregate/generalized only).**
- **Observer PII** (names, emails, usernames tied to identity, precise home coordinates, device IDs)
  is detected and **removed or pseudonymized** by default; outputs carry no living-individual PII.
  Consistent with Elyos's guardrail, identity-level data is handled as deceased/aggregate/generalized
  only — we de-identify rather than retain.
- **Conservation-sensitive locations** (poachable, collectible, or disturbance-sensitive species —
  rare orchids, raptor nests, herpetofauna, charismatic megafauna) are detected against a
  **sensitivity source** (IUCN status, national/partner sensitive-species lists) and
  **generalized**: coordinates coarsened to an agreed grid/centroid, dates coarsened, exact-locality
  text suppressed. This follows established practice (GBIF sensitive-species generalization). Default
  is fail-closed: unknown sensitivity ⇒ generalize.
- **PII / sensitivity detection methodology** (repeatable, recorded in the gate artifact):
  column-name + value heuristics for direct identifiers; quasi-identifier / k-anonymity check
  (flag smallest equivalence class below **k ≥ 5**); geo-precision threshold (flag finer than the
  partner's stated aggregation, default coarsen beyond ~3 decimal places unless justified);
  sensitivity-list lookup by resolved taxon. Any flag ⇒ suppress/generalize; the methodology output
  (which checks ran, what fired) is committed.

**Provenance & attribution.**
- Every input records source URL, publisher/platform, retrieval timestamp, version, license id +
  URL + a captured license snapshot (committed copy + SHA-256 + Wayback URL), and the required
  attribution string. Every output inherits provenance + the full run manifest. Attribution travels
  through cleaning into the packaged output; OSM and CC-BY attributions are never dropped.

**Output licensing.**
- **Code: MIT.** **Rule packs, pipeline specs, docs, datasheets: CC-BY-4.0.** **Processed data
  outputs:** license **follows the source** (CC0→CC0, CC-BY→CC-BY, ODbL→ODbL, NC stays NC); we never
  relax a source license. The packaged output records the governing license explicitly.

**Non-partisan guardrail.** Biodiversity/environmental citizen science is non-partisan by nature.
Should any civic/governance-flavored citizen-science use arise (e.g. participatory monitoring of
public services), it is **out of scope** for v1 and may only be considered later under strictly
neutral framing and separate governance review.

**Compliance gate:** no source may enter a pipeline, and no output may be packaged, until a
license/PII/sensitivity review (see Quality gates) signs off, recorded in the repo.

## Quality, review & risk gates

**Risk tier: medium** — domain accuracy (taxonomy, ecology), licensing interpretation, and privacy
judgement. **One surface is escalated to a high-stakes bar:** conservation-sensitive-species
location handling can cause real-world harm and therefore **requires credentialed
conservation/data-steward expert sign-off** before any partner relies on it (treated like a `high`
gate for that feature, even though the overall project is `medium`).

**Required reviews before a deed is "done":**
- **Code review** — maintainer review; `pnpm build && pnpm test && pnpm lint` green; DCO sign-off;
  changeset for user-facing changes; the four core invariants (determinism, non-destructive,
  privacy fail-closed, provenance-complete) covered by passing tests.
- **License + PII/sensitivity reviewer** (mandatory, every source & every packaged output) —
  confirms the source license permits reuse/derivatives, that no living-individual PII remains, and
  that sensitive-location generalization is applied correctly. Hard, non-skippable gate.
- **Domain/expert reviewer** — for validation rule packs touching taxonomy/ecology, and **mandatory
  credentialed conservation/data-steward sign-off** for any sensitive-species generalization rule
  pack or any dataset containing sensitive taxa. This is the high-stakes gate.
- **Reproducibility check** — every delivered pipeline must pass the automated double-run
  byte-identical replay; a result is not deliverable without it.
- **Partner sign-off** — a cleaned/validated output is only "delivered" when the partner project
  formally accepts and *uses* it.

**Definition of Shipped (project-level):** an open-source toolkit that has been **used by a real
citizen-science project to clean/validate/de-identify a real dataset**, where the output is
**accepted and used** by that project, the run is **reproducible** (independent replay verified),
provenance is published, and the privacy/sensitivity gate passed with expert sign-off where
sensitive taxa were present. A merged PR alone is *not* shipped.

## Roadmap & milestones

Each milestone has measurable exit criteria. M0 is a thin, honest foundation; the highest-harm
privacy surface gets its own hardened milestone (M2) before any labeling/adapter work touches real
sensitive data; partner-dependent value lands in M4. **Partner outreach runs in parallel from M0**
(long-lead) and stays `TO BE SECURED` until the Dependencies "secured" criteria are met.

**M0 — Foundation & policy (cold-start).**
Goal: prove the spine end-to-end on synthetic/openly-licensed sample data and lock the governance
constraints in writing. Exit criteria: monorepo + `@csp/spec` schemas merged; `@csp/engine` runs a
trivial deterministic pipeline with a run manifest; the **double-run byte-identical replay** test
passes in CI; the **license/PII/sensitivity policy** + the **non-destructive & reproducibility
charter** committed; CI green (build/test/lint). Partner-outreach kickoff begins here in parallel.

**M1 — Validation engine + standards I/O.**
Goal: real, reproducible validation against community standards. Exit criteria: `@csp/validate`
covers schema/range/enum/temporal/spatial/uniqueness/cross-field + taxonomic-name resolution check;
Darwin Core + Frictionless Data Package parse-in/pack-out via one adapter; `ValidationReport` emits
before/after error-rate; golden-fixture tests (valid + deliberately malformed) pass; one reference
pipeline runs on an **openly-licensed sample** end-to-end and reproduces byte-identically.

**M2 — Cleaning (non-destructive) + privacy & sensitive-data module (hardened).**
Goal: the highest-harm surface, built and expert-reviewed before any real sensitive data. Exit
criteria: `@csp/clean` non-destructive transforms with a complete change-ledger (originals
unchanged, verified by content-hash test); `@csp/privacy` PII detection/removal +
sensitive-location generalization, **fail-closed** (invariant test: no finer-than-permitted
coordinate, no denylisted PII field in any output); PII/sensitivity detection methodology codified
as a reviewable gate artifact; **conservation/data-steward expert sign-off obtained** on the
generalization rule pack before it is marked usable.

**M3 — Labeling/consensus + first real platform adapter.**
Goal: human verification + real-platform integration. Exit criteria: `@csp/label` assisted labeling
+ multi-annotator consensus with calibration items and label provenance; one real
`@csp/adapter-*` against an openly-licensed platform/export (license + provenance + sensitivity
review signed off); end-to-end pipeline (validate→clean→deidentify→label→export) reproducible on a
real openly-licensed dataset (not yet partner-accepted).

**M4 — Partner pilot + closed loop (the deed).**
Goal: deliver real value. Exit criteria: **partner project secured** (data-use agreement); the
toolkit used on the partner's **real dataset**; cleaned/validated/de-identified output **formally
accepted and used** by the partner; reproducibility independently verified; provenance published;
sensitive-handling expert sign-off recorded where applicable; Definition of Shipped met. (Hard
dependency: partner — `TO BE SECURED`; the fallback in Dependencies delivers a weaker public good
but does **not** satisfy Definition of Shipped.)

**M5 — Sustain & scale (post-delivery).**
Goal: keep it healthy and broaden. Exit criteria: a second adapter/domain or a second pilot; ops +
contribution runbook; rule-pack governance/versioning process; outcome-tracking dashboard;
maintenance rotation in place.

Dependencies: M1 depends on M0 spec/engine; M2 privacy depends on M0/M1 model + the policy; M3
labeling/adapter depends on M1 validation + M2 privacy (no real sensitive data flows before M2 is
done); M4 depends on M1–M3 and the parallel partner track.

## Work breakdown

The itemized, schema-mapped backlog lives in **`TASKS.md`** — ~18 tasks across M0–M5, each mappable
to an Elyos Task JSON, with per-milestone task tables (`ID | Title | Type | Size | Risk |
Deliverable | Depends on | Reviewer`), acceptance criteria for the most important tasks, a
Definition of Done per milestone, a future backlog, and a complete, schema-valid example Task JSON
for the first M0 task. This section is intentionally just the pointer.

## Governance, roles & stakeholders

- **Maintainer / Owner:** TBD — owns the repo, the spec, the rule-pack registry, and final merge.
- **Reviewers (rotation):** TBD — code review pool; at least one with data-engineering depth for the
  reproducibility invariants.
- **License + PII/sensitivity reviewer:** TBD (name `TO BE SECURED`) — mandatory, **non-skippable**
  gatekeeper for every source and every packaged output; must be filled before the first real
  dataset is processed, or pipelines halt. May rotate among ≥ 2 qualified reviewers.
- **Conservation / data-steward expert reviewer:** TBD (`TO BE SECURED`) — credentialed reviewer who
  signs off sensitive-species generalization rule packs and any dataset containing sensitive taxa
  (the high-stakes gate). Until named, no sensitive-taxa dataset may be processed for reliance.
- **Domain/taxonomy reviewer(s):** engaged for validation rule packs touching taxonomy/ecology.
- **Steward (last-mile owner):** TBD — owns the partner relationship and confirms the output is
  *accepted and used* (the "delivered" signal).
- **Partner / requestor:** `TO BE SECURED` — a named citizen-science project that adopts the toolkit.

## Dependencies & integrations

- **External standards/specs (pinned, recorded per run):** Darwin Core + Darwin Core Archive
  (TDWG), Frictionless Data Package / Table Schema, Croissant ML v1.0, W3C PROV, GBIF data-quality
  flag definitions, SPDX license identifiers. Versions are pinned and bumped only via a deliberate
  task.
- **Sensitivity sources:** IUCN Red List status, national/regional sensitive-species lists, and
  partner-supplied lists (the partner's own list governs in case of conflict). Sourcing a usable,
  openly-referenceable sensitivity list is a real dependency tracked in M2.
- **External platforms/datasets (output-only or licensed-in):** GBIF (CC0/CC-BY mediated records),
  iNaturalist exports (per-observation license), OpenStreetMap (ODbL); specific source/dataset
  `TO BE SELECTED` with the partner. eBird and proprietary databases are excluded unless the partner
  supplies an explicit agreement.
- **Data partner:** `TO BE SECURED` (critical path for M4). A partner counts as **"secured"** only
  when **all** of: (a) a signed **data-use agreement** (covers license, permitted derivatives,
  sensitive-species handling, attribution, retraction); (b) a **named accountable contact**; and
  (c) an **acceptance test passes** — the partner formally accepts and uses a cleaned/validated
  sample output against agreed rules. Until all three hold, the relationship stays `TO BE SECURED`.
- **Fallback if no partner is secured:** apply the toolkit to a fully openly-licensed public
  citizen-science dataset (e.g. CC0 GBIF subset), publish the cleaned output + reproducible manifest
  + datasheet openly under the source license, and commission an **independent expert spot-check** of
  the result. This exercises the full loop and delivers a public open-data good, but is explicitly
  weaker than a partner adoption and does **not** by itself satisfy Definition of Shipped.
- **Elyos pieces:** Task JSON schema (`packages/schema`), the donated-lane CLI workspace/PR flow
  (`packages/cli`), the good-deed definition + refusal guardrails. No funded-lane/runner dependency
  (this project is donated lane).

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|
| No partner project secured → toolkit built but never used (infra, not deed) | High | High | Partner outreach **in parallel from M0**; explicit "secured" criteria; openly-licensed-dataset fallback; `verifiedNeed:false` + `TO BE SECURED` everywhere | Steward |
| Sensitive-species coordinates leaked → poaching/collection/disturbance harm | Medium | Critical | Privacy fail-closed invariant; generalization to grid/centroid; sensitivity-list lookup; **conservation-expert sign-off** before reliance; default-coarsen on doubt | Conservation expert reviewer |
| Living-individual PII leaked in output | Medium | High | PII detection + removal/pseudonymization; k-anonymity check; fail-closed; License+PII reviewer gate; invariant test on outputs | License+PII reviewer |
| Misclassifying a source license (treating restricted data as open) | Medium | High | Mandatory license reviewer; license snapshot + URL; no scraping restricted platforms; default exclude on ambiguity | License+PII reviewer |
| Cleaning corrupts/alters scientifically meaningful values | Medium | High | Non-destructive change-ledger; originals preserved (content-hash test); domain review of rule packs; cleaning never imputes/fabricates | Domain reviewer |
| Pipeline non-reproducible (hidden non-determinism) | Medium | High | Double-run byte-identical CI gate; ban unseeded RNG/wall-clock/locale-dependent ops; pinned tool versions in manifest | Maintainer |
| Taxonomic-name resolution errors flag valid records or pass invalid ones | Medium | Medium | Validate against an authoritative backbone; severities (warn vs error); domain review; never auto-delete flagged records | Domain reviewer |
| ODbL share-alike obligation dropped on OSM-derived output | Low | High | Keep OSM-derived layers separable; output license follows source; attribution preserved through pipeline; reviewer check | License+PII reviewer |
| Scope creep into scientific interpretation/modeling | Medium | Medium | Explicit non-goal; reviewers reject interpretation features | Maintainer |
| Mixed-license dataset relicensed too permissively | Medium | High | Per-record license tracking; most-restrictive governs packaged output; never relax a license | License+PII reviewer |
| Maintainer burnout / abandonment post-M4 | Medium | High | Maintenance rotation, runbook, outcome dashboard before declaring done | Maintainer |

## Security & privacy

- **Threat surface:** the toolkit processes others' data locally / in the partner's CI; there is no
  hosted service in v1. Main surfaces are (a) leakage of PII/sensitive locations into outputs, logs,
  or committed fixtures, and (b) supply-chain risk in dependencies.
- **Privacy fail-closed (the core stance):** uncertain PII/sensitivity ⇒ generalize harder or halt;
  outputs are invariant-tested for no finer-than-permitted coordinates and no denylisted PII fields.
  Sensitive-species generalization defaults to coarse and is expert-signed-off before reliance.
- **No PII/sensitive data in repo or CI:** test fixtures are **synthetic or trivially public** only;
  real inspected data lives in the operator's ephemeral local scratch and is never committed to the
  repo, CI artifacts, logs, or receipts. A CI check rejects committed coordinate/PII-shaped fixtures
  above the synthetic allowlist.
- **Secrets handling:** the toolkit needs no credentials by default. If a partner platform token is
  ever required for a declared input fetch, it is supplied by the human operator and **never** written
  to logs, manifests, receipts, or committed files (per Elyos rule).
- **Data integrity & provenance:** content-addressed inputs/outputs; signed/append-only run manifests
  optional for partner handoff; corrections issued as new versioned runs (never silent in-place edits).
- **Abuse/misuse prevention:** refuse and flag any task that would steer the toolkit toward
  de-anonymizing contributors, *un-generalizing* sensitive locations, scraping restricted databases,
  or laundering a non-open dataset as open.
- **Supply chain:** pinned deps + lockfile + license-check in CI to keep dependencies permissive and
  reproducible.

## Sustainability & maintenance

- **Post-delivery ownership:** maintainer + reviewer rotation keep the toolkit and rule packs current
  with standards drift (DwC/Frictionless/Croissant versions); the steward keeps partner relationships
  and periodic re-runs alive.
- **Rule-pack governance:** validation/privacy rule packs are versioned, reviewed, and changelogged;
  sensitive-species generalization changes always re-trigger expert sign-off.
- **Outcome tracking:** a dashboard records pilots delivered/used, datasets cleaned, before/after
  error reduction, reproducibility-replay pass rate, and privacy-gate status — these (not stars or
  downloads) define health.
- **Cost:** near-zero — a local/CI toolkit with no runtime service; sustainability is about keeping
  maintenance lightweight and grant/donation-friendly.
- **Continuity:** reproducible builds, documented adapter + rule-pack interfaces, and an ops runbook
  so new platforms/projects can be added by future contributors.

## Open questions

- **Which first partner/domain** — biodiversity (DwC/GBIF), community environmental monitoring, or
  museum/herbarium digitization? Each has different sensitivity and standards needs.
- **Executor language** — is a TypeScript reference executor sufficient given the Python/R-heavy
  ecosystem, or do we need a Python executor for the spec earlier than M5? (Spec stays
  language-neutral regardless.)
- **Sensitivity source of record** — which sensitive-species list(s) are authoritative and openly
  referenceable, and how do partner-specific lists override them?
- **Generalization defaults** — default grid resolution (e.g. 1 km vs 10 km), date granularity, and
  when a record is suppressed entirely vs. generalized; must be set *with* the conservation expert.
- **Taxonomic backbone** — which authoritative checklist (GBIF backbone, Catalogue of Life) for
  name-resolution validation, and how to version it for reproducibility.
- **NC-licensed data** — exact policy boundaries for processing CC-BY-NC observations for a
  non-commercial partner; default exclude/escalate on ambiguity.
- **Consensus algorithm** for labeling — weighted majority for v1; when (if) to move to a
  Dawid–Skene/EM estimator given cold-start sparsity.
- **What counts as a verifiable "reuse" / "used by partner"** event for the outcome metric.

## References

- Elyos work rules — `C:\code\elyos\CLAUDE.md`
- Good Deed Definition + risk tiers — `C:\code\elyos\docs\good-deed-definition.md`
- Task JSON schema — `C:\code\elyos\packages\schema\src\schemas.ts`
- Portfolio roadmap — `C:\code\elyos\planning\ROADMAP.md`
- Backlog — `planning/projects/citizen-science-pipelines/TASKS.md`
- Standards (context, not dependencies to vendor): Darwin Core / Darwin Core Archive (TDWG),
  Frictionless Data (Data Package / Table Schema), Croissant ML, W3C PROV, GBIF data-quality flags
  and sensitive-species generalization guidance, IUCN Red List, SPDX, OpenStreetMap ODbL.
- Prior art (context): GBIF data-quality tooling, iNaturalist, Zooniverse, eBird (restricted-terms,
  excluded from scraping).

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified against the first draft and have been
**applied** to the plan above (and to `TASKS.md`). Each lists what changed and why.

1. **Reproducibility promoted to a tested invariant.** Added the double-run byte-identical replay as
   a CI gate (Core invariants #1; M0/M1 exit; `csp-engine-002`), not an aspiration.
2. **Non-destructive cleaning made enforceable.** Added the `ChangeLedgerEntry` model + an
   input-content-hash-unchanged test (Core invariant #2; `csp-clean-008`).
3. **Privacy fail-closed as a hard invariant.** Specified that uncertainty ⇒ coarsen/halt, with an
   output-precision/PII-denylist invariant test (Core invariant #3; M2 exit; `csp-privacy-009`).
4. **Sensitive-species harm surfaced explicitly.** Elevated poaching/collection/disturbance risk to
   a Critical-impact row and gave it a dedicated, expert-gated module and milestone (M2).
5. **High-stakes escalation within a medium project.** Defined conservation/data-steward
   credentialed sign-off as a `high`-style gate for the sensitive-handling feature specifically.
6. **k-anonymity threshold made concrete.** Added k ≥ 5 quasi-identifier check and a default
   geo-precision coarsening rule (~3 decimal places) to the detection methodology.
7. **ODbL share-alike spelled out.** Added the OSM-derived-layer separability + ODbL output +
   attribution rule and a dedicated risk row.
8. **Mixed-license handling defined.** Added per-record license tracking with "most-restrictive
   governs the packaged output" so datasets aren't relicensed too permissively.
9. **No-scraping of restricted platforms made explicit.** Named eBird/proprietary DBs as excluded
   unless the partner supplies an agreement; added to non-goals and licensing.
10. **Output license follows source.** Clarified that processed-data outputs never relax the source
    license (CC0→CC0, NC stays NC), distinct from MIT code / CC-BY rule packs.
11. **Language-neutral spec decision recorded.** Addressed the Python/R-ecosystem reality by making
    the pipeline spec the durable artifact and standards-conformant output the integration path.
12. **Standards alignment concretized.** Pinned Darwin Core, Frictionless, Croissant, W3C PROV, GBIF
    quality flags, SPDX — and required they be recorded per run for reproducibility.
13. **Provenance-complete export gate.** Added Core invariant #4: no export without a full manifest +
    per-output license, test-enforced.
14. **Cold-start fallback made real.** Added the openly-licensed-public-dataset fallback (CC0 GBIF
    subset) + independent expert spot-check, explicitly weaker than partner adoption.
15. **"Secured partner" defined objectively.** DUA + named contact + acceptance-test, mirroring the
    house pattern, so `TO BE SECURED` has a crisp exit.
16. **Outcome metrics made externally verifiable.** Required written confirmation / published archive
    / merged change; excluded self-reported reuse and vanity metrics.
17. **Error-reduction metric made fair.** Measured per-dataset against its own recorded intake
    baseline, never across datasets.
18. **Fabrication/imputation banned.** Added explicit non-goal: cleaning corrects format/dupes; it
    never invents or imputes scientific observations.
19. **Interpretation banned.** Added explicit non-goal against species-distribution modeling /
    rankings / trend conclusions to keep scope as tooling.
20. **Non-partisan civic carve-out.** Added the guardrail that governance/election citizen-science is
    out of scope unless strictly neutral and separately governed.
21. **No PII/sensitive data in repo or CI.** Added synthetic-only fixtures rule + a CI check
    rejecting coordinate/PII-shaped committed fixtures.
22. **Sensitivity source as a tracked dependency.** Made sourcing an authoritative, openly-referencable
    sensitive-species list an explicit M2 dependency/open question, with partner override.
23. **Taxonomic backbone reproducibility.** Required pinning/versioning the authoritative checklist
    (GBIF backbone / Catalogue of Life) so name-resolution validation is reproducible.
24. **Severities, not deletion.** Validators emit error/warn/info and never auto-delete flagged
    records; humans/partners decide — added to rule design and risk table.
25. **Privacy-before-platform sequencing.** Reordered milestones so the hardened privacy module (M2)
    is complete before any real platform adapter (M3) can touch real sensitive data.

## Review sign-off

**Reviewed for completeness and correctness against `PLAN_SPEC.md`, `CLAUDE.md`, the good-deed
definition, the Task schema, and the portfolio roadmap.**

- **Section completeness:** all 17 required H2 sections present and in the specified order. ✓
- **Metadata header:** Status/Version/Last updated (2026-06-28)/Owner TBD/Lane donated present. ✓
- **Honesty / `TO BE SECURED`:** no partner is invented; partner, License+PII reviewer, conservation
  expert, sensitivity source, and first dataset are all marked `TO BE SECURED`/TBD; tasks carry
  `verifiedNeed:false`. ✓
- **Guardrails:** license/provenance (open/PD/CC only; ODbL share-alike honored; no proprietary-DB
  scraping); privacy/PII (no living-individual PII; aggregate/generalized; sensitive-location
  fail-closed); non-partisan carve-out; expert sign-off for the high-stakes surface — all addressed. ✓
- **Outcome metrics:** beneficiary-centric with baselines + targets; vanity metrics excluded. ✓
- **Schema fit:** `TASKS.md` maps every task to the Task JSON schema fields; the example JSON
  validates against `packages/schema/src/schemas.ts` (all required fields present, enums correct,
  donated lane so no `fundedBudgetUsd` needed). ✓
- **Risk tier:** medium overall, with the sensitive-species surface escalated to credentialed-expert
  sign-off, consistent with the good-deed risk tiers. ✓

**Corrections made during review:** (a) clarified that the privacy module (M2) must complete before
any real platform adapter (M3) processes real sensitive data, and aligned `TASKS.md` dependencies
accordingly; (b) made the openly-licensed-dataset fallback explicitly *insufficient* for Definition
of Shipped to avoid a false "delivered" signal; (c) ensured the example Task JSON omits
`fundedBudgetUsd` (donated lane) and uses only schema-valid enum values.

**Outstanding items requiring a human decision** (also in Open questions): first partner/domain
selection; authoritative sensitivity list + generalization defaults (needs the conservation expert);
taxonomic backbone choice; and whether a Python executor is needed before M5.

**Status: ready for maintainer review.**
