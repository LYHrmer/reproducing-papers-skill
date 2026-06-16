---
name: reproducing-papers
description: Use when asked to reproduce, implement, or port a research paper's method — either into an existing codebase OR from scratch in a new project — e.g. "参考 XX 论文添加…", "复现这篇论文", "从零实现这篇论文的方法", "implement the algorithm from this paper", "port this method into our framework". Keywords 论文复现, reproduce paper, implement from paper, port method, baseline reproduction, from scratch / greenfield.
---

# Reproducing Papers

Reproducing a paper is NOT "implement what the abstract describes." It is four disciplines in order: **pin the mechanism → plan with a checklist → implement completely → validate by seeing it.** Skipping to code is the default failure.

The biggest mistake is **designing before you know what the paper changes and where it hooks into your code.** A paper states its mechanism in its own vocabulary; your job is to translate that into a precise change at a specific call site in YOUR codebase. The first, obvious interpretation is usually wrong — and you won't notice until you've built the wrong thing.

## 1. Pin the mechanism — before any design

Write ONE sentence and confirm it with the user before designing:

> "This method changes **[what entity/operation]** at **[which call site/data structure in our code]**, to **[what effect]**."

These disambiguating questions are where reproductions go wrong:

| Question | Why it matters |
|----------|----------------|
| What entity does it operate on? (observation vs landmark vs residual vs frame vs sample) | Filtering observations and filtering the things that own them are different code at different call sites. Name the wrong entity and the whole implementation is wrong. |
| Which existing function does it intercept, replace, or wrap? | If you can't point to a concrete call site, you don't understand the method yet. |
| Same result faster, or a different result? | An acceleration (index/cache) and a behavior change (fewer/different inputs) are opposite kinds of change. Don't mistake one for the other. |
| What bound or invariant does it enforce? | The contribution is usually a bound ("≤ N per cell per round"). State it. |

If your one-sentence summary turns out wrong, re-read the paper's **method** section — don't patch the design around a wrong premise. Watch for abstracts that contradict themselves (e.g. "reservoir sampling" = random retention vs. "prefer older/confirmed" = priority); the method section settles it.

**Greenfield (from scratch, no existing code)?** There's no call site to point at — instead name the smallest module interface that exercises the mechanism, and let the paper's components be your module boundaries. Every other question (entity, behavior-vs-speed, the bound) still applies.

## 2. Plan with invariants + a complete checklist

Before coding, write a plan doc (see `templates/plan_template.md`) with:
- **Core invariant(s)** — a property that must ALWAYS hold (e.g. "entry is live ⟺ it has a valid registration"), and how it's kept: atomic binding on add, one unified hook on remove.
- **Key design decisions + rationale** — sort key, eviction policy, tie-breakers (one line each).
- **Known trade-offs** — recorded up front.
- **A COMPLETE file × change checklist** — this list is your definition of done.

For a big multi-component paper, scope the iteration explicitly: pick the components you'll build now; anything you defer goes in the trade-offs table **with rationale**, never silently dropped. The checklist is the definition of done *for this iteration*; deferred ≠ forgotten.

## 3. Implement against the checklist — completely

Implement every item. When you think you're done, **diff the plan against what you actually changed.** The classic defect is silently skipping items: a config param never wired into the options struct, the manager never constructed, a value never written to the config file.

## 4. Synchronize ALL lifecycle paths

A stateful structure lives through create → update → destroy. Enumerate every path and sync state on each: add/register; value-or-position update (may cross a boundary → migrate buckets); remove via normal deletion; remove via outlier rejection; remove via marginalization/eviction. Prefer **one removal callback** over editing each site separately — it covers all paths with zero extra scanning. Watch for "state only grows" bugs: stale empty entries left on remove/update, implicit creation via `operator[]`/defaultdict access, counters that increment but never decrement. (A *bounded* monotonic cache is fine — the bug is *unbounded* growth tied to iterations or wall-clock.)

## 5. Validate by seeing it

Add visualization or instrumentation so the effect is observable on real data — this is what confirms correctness AND lets you tune parameters afterward. Overlay the structure (grid/boxes/regions) and live stats (counts, rejections, dups). **Scope it to the CURRENT frame/state, not all-history** — drawing every entry ever created makes the count grow forever and hides the real behavior. "It builds" and "tests pass" are not validation; you haven't validated until you've SEEN the method act on real input.

## Failure Modes

When you catch yourself thinking any of these, STOP and follow the Fix.

| Symptom | Root Cause | Fix | Phase |
|---------|-----------|-----|:-----:|
| "The paper obviously means X" | Skipped mechanism pinning — assumed meaning | Write ONE sentence confirming the mechanism; re-read the **method** section | P1 |
| "this is basically an index for speed" | Confused behavior change with acceleration | Answer: same result faster, or a different result? If different → it's not "just an index" | P1 |
| "I think that's everything" | No checklist — forgot items