# Deep Research Methodology: 8-Phase Pipeline

## Overview

This document contains the detailed methodology for conducting deep research. The 8 phases represent a comprehensive approach to gathering, verifying, and synthesizing information from multiple sources.

---

## Phase 1: SCOPE - Research Framing

**Objective:** Define research boundaries and success criteria

**Activities:**
1. Decompose the question into core components
2. Identify stakeholder perspectives
3. Define scope boundaries (what's in/out)
4. Establish success criteria
5. List key assumptions to validate
6. Identify the target audience and their technical level. This determines writing register, assumed knowledge, and depth of explanation.

**Ultrathink Application:** Use extended reasoning to explore multiple framings of the question before committing to scope.

**Output:** Structured scope document with research boundaries

---

## Phase 2: PLAN - Strategy Formulation

**Objective:** Create an intelligent research roadmap

**Activities:**
1. Identify primary and secondary sources
2. Map knowledge dependencies (what must be understood first)
3. Create search query strategy with variants
4. Plan triangulation approach
5. Estimate time/effort per phase
6. Define quality gates

**Graph-of-Thoughts:** Branch into multiple potential research paths, then converge on optimal strategy.

**Output:** Research plan with prioritized investigation paths

---

## Phase 3: RETRIEVE - Parallel Information Gathering

**Objective:** Systematically collect information from multiple sources using parallel execution for maximum speed

**CRITICAL: Execute ALL searches in parallel using a single message with multiple tool calls**

### Query Decomposition Strategy

Before launching searches, decompose the research question into 5-10 independent search angles:

1. **Core topic (semantic search)** - Meaning-based exploration of main concept
2. **Technical details (keyword search)** - Specific terms, APIs, implementations
3. **Recent developments (date-filtered)** - What's new in last 12-18 months (use current date from Step 0)
4. **Academic sources (domain-specific)** - Papers, research, formal analysis
5. **Alternative perspectives (comparison)** - Competing approaches, criticisms
6. **Statistical/data sources** - Quantitative evidence, metrics, benchmarks
7. **Industry analysis** - Commercial applications, market trends
8. **Critical analysis/limitations** - Known problems, failure modes, edge cases

### Parallel Execution Protocol

**Step 0: Get the current date**

Before ANY searches, retrieve today's date using Bash: `date +%Y-%m-%d`
Use the returned year for all date-filtered queries and recency checks. Do NOT assume a year from training data.

**Step 1: Launch ALL searches concurrently (single message)**

**CRITICAL: Use correct tool and parameters to avoid errors**

Choose ONE search approach per research session:

**Option A: Use WebSearch (built-in, no MCP required)**
- Standard web search with simple query string
- Parameters: `query` (required)
- Optional: `allowed_domains`, `blocked_domains`
- Example: `WebSearch(query="quantum computing 2025")`

**Option B: Use Exa MCP (if available, more powerful)**
- Advanced semantic + keyword search
- Tool name: `mcp__Exa__exa_search`
- Parameters: `query` (required), `type` (auto/neural/keyword), `num_results`, `start_published_date`, `include_domains`
- Example: `mcp__Exa__exa_search(query="quantum computing", type="neural", num_results=10)`

**Option C: Use search-cli (if installed, multi-provider)**
- Unified CLI aggregating Brave, Serper, Exa, Jina, and Firecrawl
- Install: `brew tap 199-biotechnologies/tap && brew install search-cli`
- Requires API keys: `search config set keys.[provider] YOUR_KEY`
- Auto-detects best provider per query type (academic, news, general, people)
- JSON output for structured processing: `search "query" --json`
- Modes: general, news, academic, scholar, patents, people, images, extract, scrape
- Example: `search "quantum computing 2025" -m academic --json -c 15`
- **First-time setup:** Ask user if they want to install search-cli and configure API keys


**NEVER mix parameter styles** - this causes "Invalid tool parameters" errors.

**Step 2: Spawn parallel deep-dive agents**

Use Task tool with general-purpose agents (3-5 agents) for:
- Academic paper analysis (PDFs, detailed extraction)
- Documentation deep dives (technical specs, API docs)
- Repository analysis (code examples, implementations)
- Specialized domain research (requires multi-step investigation)

**Sub-agent output format:** Require all sub-agents to return structured evidence, not free text:
```json
{"claim": "specific claim text", "evidence_quote": "exact quote from source", "source_url": "https://...", "source_title": "...", "confidence": 0.85}
```
This prevents synthesis fatigue when merging results from 3-5 agents.

**Example parallel execution (using WebSearch):**
```
[Single message with multiple tool calls]
- WebSearch(query="quantum computing 2025 state of the art")
- WebSearch(query="quantum computing limitations challenges")
- WebSearch(query="quantum computing commercial applications [CURRENT_YEAR]")
- WebSearch(query="quantum computing vs classical comparison")
- WebSearch(query="quantum error correction research", allowed_domains=["arxiv.org", "scholar.google.com"])
- Task(subagent_type="general-purpose", description="Analyze quantum computing papers", prompt="Deep dive into quantum computing academic papers from [CURRENT_YEAR], extract key findings and methodologies")
- Task(subagent_type="general-purpose", description="Industry analysis", prompt="Analyze quantum computing industry reports and market data, identify commercial applications")
- Task(subagent_type="general-purpose", description="Technical challenges", prompt="Extract technical limitations and challenges from quantum computing research")
```

**Example parallel execution (using Exa MCP - if available):**
```
[Single message with multiple tool calls]
- mcp__Exa__exa_search(query="quantum computing state of the art", type="neural", num_results=10, start_published_date="[use current year from Step 0]")
- mcp__Exa__exa_search(query="quantum computing limitations", type="keyword", num_results=10)
- mcp__Exa__exa_search(query="quantum computing commercial", type="auto", num_results=10, start_published_date="[use current year from Step 0]")
- mcp__Exa__exa_search(query="quantum error correction", type="neural", num_results=10, include_domains=["arxiv.org"])
- Task(subagent_type="general-purpose", description="Academic analysis", prompt="Analyze quantum computing academic papers")
```

**Step 3: Collect and organize results**

As results arrive:
1. Extract key passages with source metadata (title, URL, date, credibility)
2. Track information gaps that emerge
3. Follow promising tangents with additional targeted searches
4. Maintain source diversity (mix academic, industry, news, technical docs)
5. Monitor for quality threshold (see FFS pattern below)

### First Finish Search (FFS) Pattern

**Adaptive completion based on quality threshold:**

**Quality gate:** Proceed to Phase 4 when source threshold reached:
- **Quick mode:** 10+ sources with avg credibility >60/100
- **Standard mode:** 15+ sources with avg credibility >60/100
- **Deep mode:** 25+ sources with avg credibility >70/100
- **UltraDeep mode:** 30+ sources with avg credibility >75/100

**Time gate (hard stop):** If the time limit is reached before the source threshold (Quick: 2 min, Standard: 5 min, Deep: 10 min, UltraDeep: 15 min):
1. Document how many sources were found and the shortfall vs. threshold
2. Note which claims have fewer than 3 supporting sources
3. Continue to Phase 4 — do not loop back to search again

**Continue background searches:**
- If threshold reached early, continue remaining parallel searches in background
- Additional sources used in Phase 5 (SYNTHESIZE) for depth and diversity
- Allows fast progression without sacrificing thoroughness

### Quality Standards

**Source diversity requirements:**
- Minimum 3 source types (academic, industry, news, technical docs)
- Temporal diversity (mix of recent 12-18 months + foundational older sources)
- Perspective diversity (proponents + critics + neutral analysis)
- Geographic diversity (not just US sources)

**Credibility tracking:**
- Score each source 0-100 using source_evaluator.py
- Flag low-credibility sources (<40) for additional verification
- Prioritize high-credibility sources (>80) for core claims

**Techniques:**
- Use WebSearch for current information (primary tool)
- Use search-cli for multi-provider aggregated search (if installed)
- Use WebFetch for deep dives into specific sources (secondary)
- Use Exa search (via WebSearch with type="neural") for semantic exploration
- Use Grep/Read for local documentation
- Execute code for computational analysis (when needed)
- Use Task tool to spawn parallel retrieval agents (3-5 agents)

**Output:** Organized information repository with source tracking, credibility scores, and coverage map

---

## Phase 4: TRIANGULATE - Cross-Reference Verification

**Objective:** Validate information across multiple independent sources

**Activities:**
1. Identify claims requiring verification
2. Cross-reference facts across 3+ sources
3. Flag contradictions or uncertainties
4. Assess source credibility
5. Note consensus vs. debate areas
6. Document verification status per claim

### Anti-Confirmation-Bias Triage (MANDATORY)

Before following up on any unfetched search results, perform this step:

1. **State your working conclusions explicitly.** Write them out — every emerging thesis, even tentative ones.
2. **Scan ALL unfollowed search results** from Phase 3 for signals that could contradict any working conclusion.
3. **Follow contradicting threads FIRST.** They have higher information value than confirming ones. A result that challenges your thesis is more important than ten that agree with it.
4. **Triage by source authority, not by comfort:**
   - Official engineering/product blogs from the entity being researched
   - Primary sources (papers, official announcements, regulatory filings)
   - Industry press and investigative journalism
   - Practitioner blogs and case studies
   - Marketing content and SEO articles (lowest priority)
5. **Name what you're not following up on and why.** If you skip a result, state it. This prevents silent evidence burial.

**The failure mode this prevents:** Anchoring on an early thesis and then selectively following results that confirm it while ignoring contradicting signals that are sitting right there in your search results. This is the single most common deep-research error — the evidence is found but not followed.

### Contrarian Subagent (Standard/Deep/UltraDeep modes)

After stating working conclusions, spawn a dedicated contrarian research agent:

```
Agent(
  subagent_type="general-purpose",
  description="Contrarian evidence search",
  prompt="""
  You are a contrarian researcher. Your ONLY job is to find evidence
  that CONTRADICTS these working conclusions:

  [list working conclusions here]

  For each conclusion:
  1. Search specifically for disconfirming evidence
  2. Prioritise official/primary sources over commentary
  3. Look for: official announcements, engineering blog posts,
     peer-reviewed data, named practitioners with contrary results
  4. Return what you found — raw, no editorialising
  5. If you find nothing contradicting a conclusion, say so explicitly

  Do NOT try to be balanced. Do NOT confirm anything.
  Your job is to break these conclusions or fail trying.
  """
)
```

**When contrarian results return:**
- If the contrarian agent found disconfirming evidence, you MUST address it before proceeding to Phase 4.5. Fetch the sources. Update or discard the affected conclusions.
- If the contrarian agent found nothing, note this — it strengthens your confidence legitimately.
- NEVER dismiss contrarian findings because they conflict with your existing evidence pile. Volume of confirming sources does not outweigh one authoritative disconfirming source.

**Quality Standards:**
- Core claims must have 3+ independent sources
- Flag any single-source information
- Note recency of information
- Identify potential biases
- Every working conclusion must survive the contrarian check before proceeding

**Output:** Verified fact base with confidence levels, annotated with contrarian check results

---

## Phase 4.5: OUTLINE REFINEMENT - Dynamic Evolution (WebWeaver 2025)

**Objective:** Adapt research direction based on evidence discovered

**Problem Solved:** Prevents "locked-in" research when evidence points to different conclusions or uncovers more important angles than initially planned.

**When to Execute:**
- **Standard/Deep/UltraDeep modes only** (Quick mode skips this)
- After Phase 4 (TRIANGULATE) completes
- Before Phase 5 (SYNTHESIZE)

**Activities:**

1. **Review Initial Scope vs. Actual Findings**
   - Compare Phase 1 scope with Phase 3-4 discoveries
   - Identify unexpected patterns or contradictions
   - Note underexplored angles that emerged as critical
   - Flag overexplored areas that proved less important

2. **Evaluate Outline Adaptation Need**

   **Signals for adaptation (ANY triggers refinement):**
   - Major findings contradict initial assumptions
   - Evidence reveals more important angle than originally scoped
   - Critical subtopic emerged that wasn't in original plan
   - Original research question was too broad/narrow based on evidence
   - Sources consistently discuss aspects not in initial outline

   **Signals to keep current outline:**
   - Evidence aligns with initial scope
   - All key angles adequately covered
   - No major gaps or surprises

3. **Refine Outline (if needed)**

   **Update structure to reflect evidence:**
   - Add sections for unexpected but important findings
   - Demote/remove sections with insufficient evidence
   - Reorder sections based on evidence strength and importance
   - Adjust scope boundaries based on what's actually discoverable

   **Example adaptation:**
   ```
   Original outline:
   1. Introduction
   2. Technical Architecture
   3. Performance Benchmarks
   4. Conclusion

   Refined after Phase 4 (evidence revealed security as critical):
   1. Introduction
   2. Technical Architecture
   3. **Security Vulnerabilities (NEW - major finding)**
   4. Performance Benchmarks (demoted - less critical than expected)
   5. **Real-World Failure Modes (NEW - pattern emerged)**
   6. Synthesis & Recommendations
   ```

4. **Targeted Gap Filling (if major gaps found)**

   If outline refinement reveals critical knowledge gaps:
   - Launch 2-3 targeted searches for newly identified angles
   - Quick retrieval only (don't restart full Phase 3)
   - Time-box to 2-5 minutes
   - Update triangulation for new evidence only

5. **Document Adaptation Rationale**

   Record in methodology appendix:
   - What changed in outline
   - Why it changed (evidence-driven reasons)
   - What additional research was conducted (if any)

**Quality Standards:**
- Adaptation must be evidence-driven (cite specific sources that prompted change)
- No more than 50% outline restructuring (if more needed, scope was severely mis scoped)
- Retain original research question core (don't drift into different topic entirely)
- New sections must have supporting evidence already gathered

**Output:** Refined outline that accurately reflects evidence landscape, ready for synthesis

**Anti-Pattern Warning:**
- ❌ DON'T adapt outline based on speculation or "what would be interesting"
- ❌ DON'T add sections without supporting evidence already in hand
- ❌ DON'T completely abandon original research question
- ✅ DO adapt when evidence clearly indicates better structure
- ✅ DO document rationale for changes
- ✅ DO stay within original topic scope

---

## Phase 5: SYNTHESIZE - Deep Analysis

**Objective:** Identify connections across sources while maintaining factual grounding

**Activities:**
1. Map relationships between concepts found across sources
2. Identify connections that no single source states explicitly — label these as synthesis, not fact
3. Build evidence hierarchies showing which claims have strong vs. thin support
4. Develop conceptual frameworks grounded in the evidence base

**Ultrathink Integration:** Use extended reasoning to explore non-obvious connections and second-order implications.

### Synthesis Gate Checklist

Do not proceed to Phase 6 until each item is checked:

- [ ] **Source-grounded claims:** Every pattern or theme claim cites the specific sources that contribute to it. "Sources [3], [7], and [12] each describe X" — not "sources consistently report X."
- [ ] **Disagreements preserved:** Where sources disagree, both sides are presented with their evidence. No tension has been resolved in favour of a clean narrative. If sources conflict, the report says so and shows both positions.
- [ ] **Scope-bounded language:** Every claim states its evidence base size and scope. "3 of 12 sources report X" — not "X is a widespread pattern." If a finding is limited to specific contexts (one industry, one region, one time period), that limitation is stated.
- [ ] **Cross-source verification:** Before attributing a shared theme to multiple sources, each source has been re-checked to confirm it actually says what is being attributed. If 2 of 5 sources mention a pattern, the report says "2 of 5" — not "sources broadly agree."
- [ ] **Fact vs. synthesis distinction:** Every statement is clearly marked as either a sourced fact ("[N] reports...") or a synthesis ("Connecting [3] and [7] suggests..."). The reader can always tell which is which.

**Output:** Synthesized analysis with per-claim source attribution, preserved disagreements, and scope-bounded language

---

## Phase 6: CRITIQUE - Quality Assurance

**Objective:** Verify factual accuracy and structural integrity of the research output

### Critique Checklist (All Modes)

Run these 5 binary checks against the Phase 5 output. Each is pass/fail — cite the specific text that passes or fails.

1. **Source-grounded synthesis:** Does every claim in the Synthesis section cite specific sources? Find each claim and check. (Pass = all claims cite sources. Fail = quote the uncited claims.)
2. **Limitations present:** Does the Limitations section contain at least 2 specific, non-generic gaps? (Pass = 2+ specific gaps named. Fail = section missing, empty, or only contains generic hedging.)
3. **Scope-bounded language:** Are there any claims using "consistently," "broadly," "widely," or similar that cite fewer than 3 sources? (Pass = all broad claims have 3+ sources. Fail = quote the over-scoped claims.)
4. **Fact vs. synthesis distinction:** Is every statement clearly marked as either a sourced fact or a synthesis? (Pass = reader can always tell which is which. Fail = quote ambiguous statements.)
5. **Contradiction preservation:** Do any contradictions surfaced in Phase 4 appear in the final output, or were they silently smoothed away? (Pass = contradictions preserved with both sides shown. Fail = identify what was smoothed.)

### Execution by Mode

**Standard mode:** Run the checklist inline. For each item, state the specific evidence (quote the text) before marking pass/fail. This evidence-before-score ordering prevents leniency bias.

**Deep/UltraDeep modes:** Spawn a separate critique agent with fresh context to prevent anchoring to prior output:

```
Agent(
  subagent_type="general-purpose",
  description="Research critique agent",
  prompt="""
  You are a factual critique agent. You receive a research report draft
  and source material. Your job is to run a binary checklist — not judge
  quality, but verify specific factual properties.

  For each check below, quote the specific text that passes or fails.
  Do not rate overall quality. Do not be lenient. Report what you find.

  [paste the 5 checklist items above]

  Report draft:
  [paste report]

  Source material summary:
  [paste verified fact base from Phase 4]
  """
)
```

**Limitation acknowledged:** This subagent is the same model family, so self-preference bias is not eliminated — but fresh context removes anchoring to the generation conversation, and binary checklist framing (pass/fail with evidence) reduces leniency bias compared to holistic quality judgment.

### Critical Gap Loop-Back

If critique identifies a critical knowledge gap (not just a writing issue), return to Phase 3 with targeted "delta-queries" before proceeding to Phase 7. Time-box to 3-5 minutes.

**Output:** Checklist results with specific evidence for each pass/fail

---

## Phase 7: REFINE - Evidence-Based Improvement

**Objective:** Address issues flagged by Phase 6 without introducing fabricated support or smoothing away disagreements

**Activities (driven by Phase 6 checklist results):**

1. **Weak arguments:** For claims Phase 6 flagged as weakly supported, either find additional evidence via targeted search (time-boxed 3 min) or downgrade the claim's confidence level. Do not fabricate support — if evidence doesn't exist, say so.
2. **Contradictions:** For contradictions Phase 6 flagged as smoothed, restore both sides with their evidence. If one side has stronger evidence, state which and why. Do not resolve the tension into a single narrative.
3. **Gaps:** For knowledge gaps Phase 6 identified, conduct targeted searches (time-boxed 3 min). If no evidence found, document the gap in the Limitations section.
4. **Clarity:** Improve writing clarity without changing factual claims or removing caveats.

### Fallback Mechanical Checks

Run these 3 checks regardless of Phase 6 output (catches issues Phase 6 may have missed due to leniency):

1. Does every Synthesis claim cite specific sources? If not, add citations or move to Limitations.
2. Does the Limitations section contain at least 2 specific gaps? If not, add them.
3. Are there any "consistently"/"broadly"/"widely" claims citing fewer than 3 sources? If so, replace the scope language with the actual count.

**Output:** Refined research with all Phase 6 failures addressed and fallback checks passed

---

## Phase 8: PACKAGE - Report Generation

**Objective:** Deliver professional, actionable research

**Activities:**
1. Structure report with clear hierarchy
2. Write executive summary
3. Develop detailed sections
4. Create visualizations (tables, diagrams)
5. Compile full bibliography
6. Add methodology appendix

**Output:** Complete research report ready for use

---

## Advanced Features

### Graph-of-Thoughts Reasoning

Rather than linear thinking, branch into multiple reasoning paths:
- Explore alternative framings in parallel
- Pursue tangential leads that might be relevant
- Merge insights from different branches
- Backtrack and revise as new information emerges

### Parallel Agent Deployment

Use Task tool to spawn sub-agents for:
- Parallel source retrieval
- Independent verification paths
- Competing hypothesis evaluation
- Specialized domain analysis

### Adaptive Depth Control

Automatically adjust research depth based on:
- Information complexity
- Source availability
- Time constraints
- Confidence levels

### Citation Intelligence

Smart citation management:
- Track provenance of every claim
- Link to original sources
- Assess source credibility
- Handle conflicting sources
- Generate proper bibliographies

---

## Critical Phase Reminders

**These rules override any conflicting instruction elsewhere in this document.**

1. **Anti-confirmation-bias triage is mandatory before following up Phase 3 results.** Before following up on any unfetched search results, state your working conclusions explicitly. Scan ALL unfollowed results for signals that could contradict any working conclusion. Follow contradicting threads FIRST — they have higher information value than confirming ones.

2. **Every synthesis claim must cite specific sources and be bounded to its evidence base.** "Sources [3], [7], and [12] each describe X" — not "sources consistently report X." If 2 of 5 sources mention a pattern, say "2 of 5" — not "sources broadly agree."

3. **Contradictions must be preserved, not resolved.** When sources disagree, present both sides with their evidence. Do not resolve tensions in favour of a clean narrative. If one side has stronger evidence, state which and why — but show both.
