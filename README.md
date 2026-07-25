# citizen-science-pipelines

> Citizen-science projects — biodiversity observation networks, community air-quality and water-quality monitoring, phenology and weather logging, transcription and astronomy classification efforts — co  ·  **Risk tier:** med  ·  **Status:** planning

Citizen-science projects — biodiversity observation networks, community air-quality and water-quality monitoring, phenology and weather logging, transcription and astronomy classification efforts — collectively generate an enormous, scientifically valuable stream of volunteer-contributed observations. That stream is also **messy by construction**: inconsistent formats, typos, unit confusion, duplicates, taxonomic name drift, implausible coordinates and dates, and — critically — inadvertent **personal data** (observer names, home coordinates) and **conservation-sensitive locations** (precise positions of poachable or collectible species). Most projects clean and validate this data with one-off, undocumented, non-reproducible scripts, or not at all. The work is repeated independently by thousands of projects and is rarely shared.

**Definition of shipped:** citizen-science project to clean/validate/de-identify a real dataset**, where the output is **accepted and used** by that project, the run is **reproducible** (independent replay verified), provenance is published, and the privacy/sensitivity gate passed with expert sign-off wher

This is a **Hee-Lee Oss** good-deed project. Contributors pull a task, do it with their own coding agent, and open a PR. Get started: https://github.com/HeeLeeOss/hee-lee-oss-downloads

## Plan
- [PLAN.md](./PLAN.md) — robust enterprise plan (vision, architecture, roadmap, risks; includes an applied-improvements appendix + review sign-off)
- [TASKS.md](./TASKS.md) — schema-mapped task backlog
- [tasks/](./tasks/) — ready-to-pull task JSON(s)

## Contribute
```bash
hee-lee-oss browse
hee-lee-oss next --repo HeeLeeOss/citizen-science-pipelines --no-fork
```

## Licensing & review
- Open license (see PLAN.md).
- Risk tier **med** — deeds are *delivered, not merged*; a domain reviewer (and expert sign-off for any high-stakes content) must approve before merge.

> Planning stage; no adopting partner secured yet (`verifiedNeed: false` on delivery-dependent tasks).
