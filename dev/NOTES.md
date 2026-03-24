# Deep Research Skill — Development Notes

This file is NOT loaded during research runs. It exists for skill development sessions — read this when modifying the skill, not when using it.

---

## Open Issue: Phase 3 Retrieval Bias (2026-03-24)

### The Problem

The skill repeatedly missed a critical primary source (Hristo Danchev's LinkedIn engineering blog post, March 12 2026, confirming 360Brew deployment) across two separate research runs on the same topic. The 10 fixes applied to Phases 4-7 (synthesis, critique, refinement) improved output quality but did not help here because the source was never retrieved in the first place.

### Root Cause

None of the 10 fixes touch Phase 3 (RETRIEVE). The model forms a working hypothesis early in retrieval, then selects which search results to fetch based on that hypothesis. A third-party blog (The Linked Blog) said "no public confirmation of deployment" — the model anchored on this and never searched for the actual primary source (LinkedIn's own engineering blog by Danchev). Classic confirmation bias during retrieval, not during analysis.

### Proposed Fix: Three Phase 3 Instructions

A subagent designed three instructions that target different points in the Phase 3 retrieval loop:

**1. Source Proximity Obligation (before each fetch round)**
Before fetching, classify each search result by distance from origin — is this the entity/researcher/project speaking, or someone commenting on them? Fetch at least one origin-tier source per round. If none appear in results, run a supplementary query targeting the origin channel before proceeding.

Generalized form: "Go upstream. If you're only finding commentary, search for the original." Works for entity research (LinkedIn's own blog), field surveys (actual journal papers vs pop-science), comparisons (each project's own docs vs third-party comparison articles).

**2. Staleness Check on Negative Claims (after each fetch, before updating working hypothesis)**
When a fetched source claims "X has not been confirmed" or "no evidence of X," record publication date. If >14 days old relative to the event timeframe, flag as STALE-NEGATIVE and run a date-bounded search covering the gap. Cannot become the working hypothesis without this check.

Generalizes cleanly — any negative claim from any source type can be outdated.

**3. Hypothesis-Adversarial Search Round (scheduled, midpoint of Phase 3 time budget)**
Pause. State working hypothesis in one sentence. Construct one search query designed to prove it wrong, using different terms than any previous query. Not a refinement — a reframe. Mandatory, cannot be skipped. Fetch top 2 results before resuming.

Already general — "state what you believe, then search for evidence you're wrong" applies to any research type.

### Open Design Question: Branching vs Generalization

These three instructions were originally designed for entity-focused research ("what did LinkedIn deploy?"). They need to generalize to field surveys ("state of narrative psychology"), comparisons ("React vs Vue"), and trend analysis.

**Arguments for branching by research type:**
- Specific instructions are more likely to be followed than general ones
- Different research types fail in genuinely different ways
- The skill already branches on mode (quick/standard/deep/ultradeep)

**Arguments for generalization (current preference):**
- The skill is already ~500 lines. Branching multiplies surface area and maintenance burden
- Research types aren't cleanly separable — many queries are simultaneously entity-focused AND comparative AND trend analysis. Misclassification adds a new failure point
- The underlying principle is general: "don't let early findings constrain your search space"
- Instruction overload risk: >15-20 discrete instructions and compliance drops

**Decision:** Try generalization first. Only branch if generalized versions fail testing. The test for whether a generalized instruction works: would it have caught the Danchev miss?

### Status

Not yet implemented. These notes capture the analysis for the next skill development session.

---

## Completed: 10 Prompt-Critique Fixes (2026-03-24)

Applied all 10 fixes from plan `starry-tinkering-kahn.md`. Files modified:
- `reference/methodology.md` — Fixes 1, 2, 3, 5, 6, 7 (Phase 5 hallucination guards, Phase 6 critique separation, Phase 7 reframing, Critical Phase Reminders at end, escape hatch behavior, Phase 1 audience)
- `SKILL.md` — Fix 8 (Autonomy Principle reframing)
- `templates/report_template.md` — Fix 9 (prose-first Executive Summary and Synthesis)

Re-critique confirmed all 10 original issues resolved. Only 3 new low-severity notes found (one stale HTML comment, fixed immediately).

Test run (LinkedIn pods research, fresh chat) showed measurable improvements in synthesis quality: debunked the fabricated "97% detection" claim, scope-bounded 360Brew deployment claims, preserved contradictions, documented evidence gaps as findings. But missed Danchev — see open issue above.
