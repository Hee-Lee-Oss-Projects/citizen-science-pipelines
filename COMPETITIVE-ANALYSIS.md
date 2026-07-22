# Competitive & Improvement Analysis — `citizen-science-pipelines`

Scope: open data pipelines/tooling to help citizen-science projects ingest, clean, validate, and
publish data (FAIR, reproducible). Lane: donated. Risk tier: medium (sensitive-species handling
escalated to high). Analysis grounded against the live ecosystem with cited sources.

Source docs reviewed: `PLAN.md` (v0.1.0, 2026-06-28), `TASKS.md` (~18 tasks, M0–M5). The plan is
unusually strong: honest `TO BE SECURED` markers, fail-closed privacy invariants, four test-enforced
core invariants, and a 25-item "improvements applied" appendix. The gaps below are therefore mostly
at the edges (ecosystem fit, sequencing, metric precision), not foundational.

---

## 1. Correctness & completeness review of PLAN.md

Concrete gaps, errors, weak metrics, and risks — emphasis on data quality/validation, FAIR +
standards, privacy/PII, attribution, interoperability, and sustainability.

**A. Standards coverage has a conspicuous hole: PPSR Core.** The plan aligns to Darwin Core,
Frictionless, Croissant, and W3C PROV but never mentions **PPSR Core**, the citizen-science
community's own transdisciplinary data+metadata standard (project, dataset, and observation models)
maintained by the Association for Advancing Participatory Sciences and used by SciStarter, CitSci.org
and the Wilson Center federal catalog ([core.citizenscience.org](https://core.citizenscience.org/),
[participatorysciences.org](https://participatorysciences.org/2015/10/09/ppsr_core-metadata-standard/)).
For a project whose headline beneficiary is *citizen-science projects* (not just biodiversity
aggregators), omitting PPSR Core is a real interoperability and credibility gap — DwC is the
biodiversity-occurrence subset, but air/water-quality and phenology projects (named in the exec
summary) are not natively DwC. **Fix:** add PPSR Core (and its DwC mapping) as a first-class
standard, or scope the exec summary down to biodiversity-occurrence data to match the DwC-centric
design.

**B. The `dwc:dataGeneralizations` / `informationWithheld` terms and the TDWG Sensitive Species
Extension are not referenced.** GBIF/TDWG best practice is explicit: when you generalize, you record
*how/why* via `dwc:dataGeneralizations`, `informationWithheld`, and `dataGeneralizations`, and there
is a dedicated **Sensitive Species Extension** under active TDWG development
([tdwg.org](https://www.tdwg.org/community/dwc/sensitive-species/),
[GBIF best-practices guide](https://www.gbif.org/document/80512/guide-to-best-practices-for-generalising-sensitive-species-occurrence-data)).
The plan's `PrivacyRule` model coarsens coordinates but does not specify that the *fact and method*
of generalization must be emitted as standard DwC terms on the output. Without that, a generalized
record is silently indistinguishable from a precise one downstream — a correctness defect for the
privacy module. **Fix:** require the privacy module to stamp `dwc:dataGeneralizations` /
`informationWithheld` and adopt the Sensitive Species Extension fields.

**C. "Byte-identical reproducibility" is over-specified and may be brittle.** The 100% byte-identical
invariant is admirable but fragile across OS/locale/library-version boundaries (float formatting,
CSV line endings, hash-map ordering, Unicode normalization). The plan bans some non-determinism
sources, but byte-identity across *independent third-party replay* (the stated metric, line 117)
typically requires a pinned container/toolchain, which v1's "runs locally / in partner CI" model does
not guarantee. **Fix:** either (a) scope byte-identity to a pinned environment (container digest in
the manifest), or (b) relax the external metric to *canonical-form-identical* (normalized output +
identical change-ledger + identical content hashes of a canonicalized representation). As written the
metric risks failing for benign reasons and inviting fudging.

**D. Validation depth is under-specified vs. existing art.** The validators listed
(schema/range/enum/temporal/spatial/uniqueness/cross-field + taxonomic) are essentially a
reimplementation of what **CoordinateCleaner** and **bdc** already do for the spatial axis
(sea/zero/centroid/capital/institution/urban/outlier flags over ~91M GBIF records)
([CoordinateCleaner](https://ropensci.github.io/CoordinateCleaner/),
[bdc](https://brunobrr.github.io/bdc/)). The plan does not say whether it will *wrap/port* these
battle-tested flag sets or rebuild them. Rebuilding spatial-validity heuristics from scratch is a
multi-year research effort the plan under-budgets (one `medium` task, `csp-validate-005`). **Fix:**
explicitly adopt CoordinateCleaner/bdc's flag taxonomy as the spec's vocabulary and treat the TS
engine as an orchestrator/standardizer, not a novel cleaner.

**E. Volunteer attribution is asserted but not modeled.** The plan repeatedly promises attribution is
"preserved through the pipeline," but the canonical data model has no `recordedBy` / contributor-credit
field, and the privacy module *removes observer identity by default* (line 306). There is an unresolved
tension: **de-identification (remove observer PII) vs. attribution (credit the volunteer)**. iNaturalist
and Zooniverse both keep contributor identity as a deliberate credit mechanism. The plan needs an
explicit policy: pseudonymous-but-stable contributor IDs, opt-in credit, or aggregate
acknowledgment — and a model field for it. As written, "attribution to volunteers" (a stated
guardrail) is in direct conflict with "outputs carry no living-individual PII." **This is a genuine
design contradiction, not just a doc gap.**

**F. Weak/ambiguous metrics.** (i) "≥50% reduction in flagged errors" is gameable — looser validators
flag fewer errors; the metric should be tied to a *fixed, versioned rule pack* baseline. (ii)
"100% byte-identical" — see C. (iii) "≥1 spec/rule pack reused externally" is a thin reuse bar for a
12-month horizon. (iv) No metric for *PPSR Core / DwC conformance pass rate* against an external
validator (e.g. GBIF data-validator) — conformance is asserted but not measured against the
authoritative tool.

**G. Sustainability/funding is hand-waved.** "Cost: near-zero" is true for compute but false for the
*scarce* resources this plan depends on: a credentialed conservation/data-steward reviewer and a
License+PII reviewer, both `TO BE SECURED`, both on the critical path, both unpaid. The biggest
sustainability risk is not maintainer burnout (listed) but **inability to recruit/retain the expert
gatekeepers** the whole privacy story rests on. Not in the risk table.

**H. Minor errors/omissions.** (i) Croissant is cited as "Croissant ML v1.0" — fine, but note its
Responsible-AI extension (**Croissant-RAI**) is the part relevant to provenance/datasheets
([MLCommons](https://mlcommons.org/2024/03/croissant_metadata_announce/)). (ii) The plan excludes
eBird raw downloads (correct — restrictive terms), but eBird *is* mediated into GBIF as the EOD
dataset under a citable license; the plan could consume the GBIF-mediated form, which it doesn't
acknowledge. (iii) k≥5 is asserted as the anonymity threshold but no l-diversity/t-closeness
consideration for sensitive *attributes* (e.g. a rare taxon is itself the quasi-identifier) — for
sensitive species the taxon, not just the coordinate, leaks location.

---

## 2. Competitive landscape

Two distinct competitor classes: **platforms** (collect + host + minimally clean data) and
**tooling/standards** (clean/validate/publish). Hee-Lee Oss's project is firmly in the second class, which
is far less crowded — but the platforms have captured the users.

**Platforms (own the data + the users):**

- **iNaturalist** — the gold standard for biodiversity observations + community ID. Has
  **research-grade** data-quality assessment and, critically, mature **geoprivacy**: automatic
  taxon-geoprivacy obscuring of at-risk species to a 0.2°×0.2° (~400 km²) cell, plus open/obscured/
  private observer controls ([geoprivacy](https://www.inaturalist.org/pages/geoprivacy),
  [help](https://help.inaturalist.org/en/support/solutions/articles/151000169938-)).
  *Strength:* enormous, already-cleaned corpus; sensitive-location handling solved at scale.
  *Weakness:* obscuring is per-platform and coarse; per-observation licenses are mixed (much CC-BY-NC);
  no reusable, auditable, cross-platform cleaning method.
- **eBird (Cornell Lab)** — 5,000+ automated submission filters + **2,000+ expert reviewers**; in 2017,
  4.6% of records flagged, ~43% of those invalidated — a human+machine learning loop
  ([eBird review process](https://support.ebird.org/en/support/solutions/articles/48000795278-the-ebird-review-process),
  [Expert Reviewer Network](https://biss.pensoft.net/article/25394/)).
  *Strength:* best-in-class domain validation. *Weakness:* closed, taxon-specific (birds), restrictive
  raw-download terms — explicitly the model Hee-Lee Oss can't copy but should learn from (the expert-loop).
- **Zooniverse** — general crowd-classification platform with documented **aggregation/consensus**
  tooling (extractors→reducers→consensus, Python CLI) ([aggregation](https://blog.zooniverse.org/2018/10/26/zooniverse-data-aggregation/),
  [classification exports](http://developer.zooniverse.org/en/latest/science/classifications_exports.html)).
  *Strength:* proven consensus algorithms — directly relevant to Hee-Lee Oss's `@csp/label`.
  *Weakness:* tied to Zooniverse's own data model; not a general FAIR-publishing pipeline.
- **CitSci.org / SciStarter / Anecdata** — project-hosting platforms. CitSci.org co-developed PPSR
  Core ([BISS](https://biss.pensoft.net/article_preview.php?id=75666)). **Anecdata** is notable for
  *purpose-built privacy features* for citizen-collected data (individual + project-level controls,
  200+ projects) ([Frontiers paper](https://www.frontiersin.org/journals/climate/articles/10.3389/fclim.2021.620100/full)).
  *Weakness:* each is a silo; cleaning/validation is minimal and non-portable.
- **GBIF** — the aggregator/standard-setter: Darwin Core, DwC-Archive, the data-validator, data-quality
  flags, and the authoritative **sensitive-species generalization guidance**
  ([GBIF DwC](https://www.gbif.org/darwin-core), [survey-data guide](https://docs.gbif.org/guide-publishing-survey-data/en/)).
  *Strength:* defines the target format Hee-Lee Oss must conform to. *Weakness:* GBIF cleans on *ingestion*
  centrally; it does not give projects a reusable upstream cleaning toolkit — exactly the gap.

**Tooling / standards (Hee-Lee Oss's real arena):**

- **Frictionless Data (Open Knowledge)** — Data Package / Table Schema + `frictionless-py` (describe/
  extract/validate/transform, unified validation report, pipelines)
  ([framework](https://framework.frictionlessdata.io/),
  [datapackage-pipelines](https://github.com/frictionlessdata/datapackage-pipelines)).
  *Strength:* mature, adopted, multi-language (py/R/js), exactly the declarative-spec philosophy
  Hee-Lee Oss espouses. *Weakness:* generic tabular data — no taxonomic/spatial/conservation-sensitivity
  semantics, no privacy module. **This is both the closest competitor and the best foundation to build
  on rather than reinvent.**
- **OpenRefine** — the de-facto data-cleaning tool; records a stepwise, exportable-as-JSON,
  re-appliable operation history (reproducible cleaning + provenance) and reconciliation against
  Wikidata/ROR; has documented GBIF-cleaning workflows via Galaxy training
  ([Galaxy GBIF/OpenRefine tutorial](https://training.galaxyproject.org/training-material/topics/ecology/tutorials/openrefine_gbif/tutorial.html),
  [TaPP provenance paper](https://www.usenix.org/conference/tapp2019/presentation/mcphillips)).
  *Strength:* the operation-history-as-reusable-recipe is *precisely* Hee-Lee Oss's "pipeline spec is the
  durable artifact" idea — already shipping. *Weakness:* GUI-centric, single-machine, no
  privacy/sensitivity, recipes are OpenRefine-specific not standard.
- **CoordinateCleaner / bdc (rOpenSci)** — automated spatial/temporal flagging for biological records
  at GBIF scale ([CoordinateCleaner](https://ropensci.github.io/CoordinateCleaner/),
  [bdc](https://brunobrr.github.io/bdc/)). *Strength:* the validated flag taxonomy Hee-Lee Oss should adopt.
  *Weakness:* R-only, no spec/provenance/privacy framing.
- **Galaxy** — reproducible scientific-workflow platform with ecology training incl. data cleaning
  ([Galaxy ecology](https://training.galaxyproject.org/training-material/topics/ecology/)).
  *Strength:* reproducibility + provenance at scale. *Weakness:* heavy, server-hosted, general-purpose.
- **Open Humans / Frictionless / PPSR Core** — Open Humans is personal-data-sharing infrastructure
  (consent-centric), tangential but relevant to the PII/consent stance.

**Net:** No incumbent combines (declarative reusable cleaning recipe) + (biodiversity-grade
validation) + (fail-closed privacy/sensitive-species generalization) + (FAIR/PROV provenance) +
(standards-out to DwC *and* PPSR Core) in one agent-neutral, portable toolkit. That is the white space.

---

## 3. Gaps we can fill

1. **A privacy/sensitive-data module as a portable, auditable library.** iNaturalist obscures, GBIF
   guides — but no one ships a *reusable, expert-signed, fail-closed* generalization rule pack that any
   project can apply and any third party can audit, emitting `dwc:dataGeneralizations`. Genuinely novel.
2. **Cross-standard publishing (DwC ⊕ PPSR Core ⊕ Frictionless ⊕ Croissant) from one spec.** The
   ecosystem is siloed by standard; a single pipeline that emits all of them is unfilled.
3. **Reproducible, provenance-complete cleaning *recipe* that is tool-neutral.** OpenRefine recipes are
   OpenRefine-bound; Frictionless pipelines are tabular-generic. A standards-aware, language-neutral,
   PROV-emitting spec is the gap.
4. **The de-identification ⊕ attribution reconciliation** (if §1E is solved) — nobody has a clean answer
   to "credit the volunteer without leaking the volunteer."
5. **A datasheet/"nutrition label" auto-generated from the run** documenting exactly what cleaning was
   applied — ties to Croissant-RAI/Datasheets; currently manual everywhere.
6. **Quality flags as portable data** rather than locked in CoordinateCleaner(R)/eBird(closed): a shared
   versioned vocabulary other tools can consume.

## 4. Differentiators to win

1. **Fail-closed, expert-gated sensitive-species generalization as a first-class, audited module** — the
   single strongest differentiator. No competitor offers a reusable, signed-off, test-invariant
   ("no coordinate finer than permitted, ever") privacy artifact emitting standard DwC withholding terms.
2. **The pipeline *spec* as the durable, shareable, reusable good** — one project's audited cleaning
   method becomes another's, versioned and PROV-stamped. (Beats OpenRefine's tool-bound recipes.)
3. **Reproducibility as a tested CI invariant**, not a promise — double-run replay gate.
4. **Radical license/provenance honesty** — per-record license tracking, most-restrictive-governs,
   ODbL share-alike honored, no scraping. A trust differentiator for partners burned by data laundering.
5. **Agent-neutral, local/CI, no-SaaS, no-lock-in** — projects keep their data; nothing is hosted.
6. **Standards-out to the citizen-science community's *own* standard (PPSR Core), not just biodiversity
   DwC** — if §1A is fixed, this widens the beneficiary base beyond biodiversity to air/water/phenology.

## 5. Claude API leverage — and hard limits

**Where Claude adds leverage (assistive, human-verified):**

1. **Cleaning/validation rule *authoring*** — draft `ValidationRule`/`CleaningTransform` specs from a
   messy sample (infer enums, ranges, regexes, unit patterns, likely typos), proposed as *reviewable
   declarative data*, never auto-applied. Claude writes the recipe; the deterministic engine runs it.
2. **Darwin Core / PPSR Core / Frictionless field mapping** — map a project's idiosyncratic columns to
   standard terms (the highest-friction manual task), with confidence + rationale per mapping for human
   sign-off. High-value, low-risk because mappings are reviewed before use.
3. **Documentation / datasheet / methodology generation** — auto-draft the "what cleaning was applied"
   datasheet, change-ledger summaries, provenance narratives, and rule-pack changelogs from the run
   manifest. Pure summarization of deterministic facts.
4. **QC triage & explanation** — cluster/triage validation flags, explain *why* a record was flagged in
   plain language, suggest (not decide) likely fixes for human review.
5. **PII/quasi-identifier *candidate* detection** — surface columns that *look* like direct/indirect
   identifiers for a human to confirm (augments, never replaces, the heuristic + k-anonymity gate).

Model/SDK specifics (ids, pricing, caching, structured tool-use for rule extraction) should be pulled
from the `claude-api` skill at implementation time, not guessed.

**Where Claude must NOT decide (hard gates — human/expert only):**

- **Scientific/taxonomic/ecological validity** — whether a flagged record is genuinely wrong, whether a
  name resolves, whether a value is plausible: domain reviewer, not Claude. (eBird's loop is *expert*
  reviewers for a reason.)
- **Sensitive-data / PII determinations** — whether a taxon/location is conservation-sensitive and how
  coarsely to generalize: credentialed conservation/data-steward sign-off, fail-closed. Claude may flag
  candidates; it may never clear a record for release.
- **No fabrication / imputation, ever** — Claude must not invent, impute, or "smooth" observations to
  fill gaps; cleaning corrects format/dupes only (an explicit plan non-goal). This is the single most
  important Claude guardrail here — LLMs' instinct to "helpfully complete" data is a fabrication risk.
- **License & attribution determinations** — whether a source permits reuse/derivatives, and the
  required attribution string: License+PII reviewer verifies; Claude's read of a license is advisory only.
- **Final publish decision** — a human at the partner publishes; nothing auto-publishes.

Design rule: **Claude proposes declarative artifacts (rules, mappings, docs); the deterministic engine
executes; humans/experts approve.** Keeping Claude out of the execution path preserves the byte-identical
reproducibility invariant (LLM calls are non-deterministic and must never run inside a pipeline step).

## 6. Ten concrete optimizations

1. **Adopt PPSR Core as a first-class standard** (model + DwC mapping) so non-biodiversity projects
   (air/water/phenology, the exec summary's own examples) are actually served. (§1A)
2. **Emit `dwc:dataGeneralizations` / `informationWithheld` + the TDWG Sensitive Species Extension** from
   the privacy module so generalized records are self-describing and standards-conformant. (§1B)
3. **Resolve the de-identify ⊕ attribute contradiction**: add a stable pseudonymous contributor-credit
   field + opt-in policy to the canonical model. (§1E)
4. **Wrap, don't rebuild, CoordinateCleaner/bdc's flag taxonomy** as the spatial-validation vocabulary;
   port their tests as golden fixtures. (§1D)
5. **Scope the reproducibility invariant to a pinned environment** (container digest in the manifest) or
   relax the external metric to canonical-form-identity. (§1C)
6. **Measure conformance against an external authority** (GBIF data-validator / Frictionless validate)
   and add a "conformance pass rate" metric, instead of self-asserting conformance. (§1F)
7. **Build a Claude-assisted "messy-CSV → draft PipelineSpec + DwC mapping" onboarding flow** — the
   highest-leverage adoption accelerator; output is reviewable declarative data, not auto-applied. (§5)
8. **Auto-generate a per-run datasheet (Croissant-RAI/Datasheets-aligned)** from the manifest as a
   standard export — turns provenance into a shippable artifact. (§3.5)
9. **Treat expert-reviewer recruitment/retention as a tracked dependency with a funded/stipend plan** —
   the real sustainability risk; add to the risk table. (§1G)
10. **Add taxon-as-quasi-identifier handling (l-diversity-style)** to the privacy module: for sensitive
    species the *taxon itself* leaks location, so suppression must consider the attribute, not only the
    coordinate. (§1H-iii)

## 7. Parallel & perpendicular spin-offs

- **Reusable FAIR-publishing toolkit (`@fair/*`)** — extract the standards-out core (DwC/PPSR-Core/
  Frictionless/Croissant packing + PROV manifest + datasheet generation) as a domain-neutral library any
  Hee-Lee Oss open-data project can depend on. The most valuable reusable asset here.
- **An MCP server** — expose `validate`, `map-to-standard`, `generate-datasheet`, and `explain-flags` as
  MCP tools so *any* agent (the donated-lane human's agent included) can call the deterministic engine and
  Claude-assisted mapping without bespoke integration. Natural fit with Hee-Lee Oss's agent-neutral core.
- **`open-data-datasheets`** — the per-run datasheet generator (§6.8) is a shared component; csp produces
  datasheets, that project standardizes/houses them. Direct, immediate tie.
- **`open-data-explainers`** — Claude-generated plain-language "why was this record flagged / what was
  done to this dataset" narratives are a shared explainer capability.
- **`open-map-gaps` / `urban-tree-inventory`** — both are spatial citizen/open-data projects that need
  exactly csp's spatial validation, coordinate-precision/privacy generalization, and DwC/ODbL provenance.
  urban-tree-inventory is a ready-made *pilot* candidate (trees: low sensitivity, public benefit,
  attribution-friendly) to de-risk the `TO BE SECURED` partner gap. open-map-gaps shares the ODbL
  share-alike machinery.
- **Sensitivity rule-pack registry** — the expert-signed generalization rule packs could be a standalone,
  citable, versioned public good consumed beyond Hee-Lee Oss (a perpendicular contribution to GBIF/TDWG practice).

## 8. Open questions

1. **PPSR Core vs. DwC-only scope** — does v1 commit to PPSR Core, or narrow the exec summary to
   biodiversity-occurrence to match the DwC-centric design? (Decides the beneficiary base.)
2. **De-identify vs. attribute** — what is the contributor-credit policy, and does the partner's DUA
   govern it per dataset?
3. **Build vs. wrap** — port CoordinateCleaner/bdc/Frictionless logic, or reimplement in TS? (Time/scope.)
4. **Reproducibility boundary** — byte-identity in a pinned container, or canonical-form identity across
   environments?
5. **First pilot** — is `urban-tree-inventory` (low-sensitivity, sibling project) the fastest path to
   close the loop and prove the toolkit before tackling sensitive taxa?
6. **Expert-reviewer economics** — how are the conservation and License+PII gatekeepers recruited,
   retained, and (if needed) compensated, given they are unpaid and on the critical path?
7. **Sensitivity source of record** — which authoritative, openly-referenceable sensitive-species list,
   and how is taxon-as-quasi-identifier risk handled beyond coordinate coarsening?
8. **Claude in onboarding** — acceptable to use Claude for spec/mapping drafting given non-determinism, as
   long as outputs are reviewed and never run inside a pipeline step? (Recommended: yes.)

---

### Sources
- PPSR Core: https://core.citizenscience.org/ · https://participatorysciences.org/2015/10/09/ppsr_core-metadata-standard/ · https://biss.pensoft.net/article_preview.php?id=75666
- GBIF/TDWG sensitive species: https://www.gbif.org/document/80512/guide-to-best-practices-for-generalising-sensitive-species-occurrence-data · https://www.tdwg.org/community/dwc/sensitive-species/ · https://docs.gbif.org/guide-publishing-survey-data/en/ · https://www.gbif.org/darwin-core
- iNaturalist geoprivacy: https://www.inaturalist.org/pages/geoprivacy · https://help.inaturalist.org/en/support/solutions/articles/151000169938-
- eBird review: https://support.ebird.org/en/support/solutions/articles/48000795278-the-ebird-review-process · https://biss.pensoft.net/article/25394/
- Zooniverse: https://blog.zooniverse.org/2018/10/26/zooniverse-data-aggregation/ · http://developer.zooniverse.org/en/latest/science/classifications_exports.html
- Frictionless: https://framework.frictionlessdata.io/ · https://github.com/frictionlessdata/datapackage-pipelines
- OpenRefine/Galaxy: https://training.galaxyproject.org/training-material/topics/ecology/tutorials/openrefine_gbif/tutorial.html · https://www.usenix.org/conference/tapp2019/presentation/mcphillips
- CoordinateCleaner/bdc: https://ropensci.github.io/CoordinateCleaner/ · https://brunobrr.github.io/bdc/
- Anecdata: https://www.frontiersin.org/journals/climate/articles/10.3389/fclim.2021.620100/full
- Croissant: https://mlcommons.org/2024/03/croissant_metadata_announce/ · https://github.com/mlcommons/croissant
