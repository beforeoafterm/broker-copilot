# Brainstorming Findings: the working record

**Date:** 2026-07-06

**What this is:** the raw thinking behind the solution, kept for anyone who wants to see the receipts.

**How to read it:** you probably don't need to. The short, polished solution is in [system-design.md](system-design.md), and how it was built is in [methodology.md](methodology.md). This is the full working record: every idea, dead end, and finding, in the order it surfaced. It is the appendix, not the pitch. Dip in if you want proof the conclusions were earned.

**Status:** Complete. Three brainstorming techniques, then a first synthesis, then a 13-agent adversarial hardening pass that found two fatal flaws and fixed them. The Hardened Synthesis section near the bottom is the authoritative version; everything before it is the journey that got there.

---

## The Problem

How do we help a broker capture a complete, sufficient business risk profile from a client in a **single call**, and package it into a compelling submission for the **right 2-3 carrier markets**, without losing the prospect to follow-up drop-off?

### Company context
The setting is an AI-native commercial insurance brokerage. Brokers talk to business owners across every industry and need to extract enough of a risk profile in one conversation to submit a strong application.

### Current approach
Per risk type: gather each carrier's question set from their appetite documents, de-duplicate overlapping questions across carriers, chain them into a decision tree, and have AI transcribe the broker's live call against that tree.

### Known failure modes
1. Building and maintaining question sets and decision trees per risk type is slow and manual.
2. The trees cannot capture every branch, because real businesses do not fit clean categories.
3. If the broker has to go back to the client after the call, the client drops out of the funnel. The full profile has to land in one pass.

### Constraint that matters
Carriers have a fixed amount of attention per application. Spraying an application to many carriers instead of the right 2-3 degrades the broker-carrier relationship over time. This is not "collect more data." It is collect the *right* data and package it into a targeted, compelling story for a small number of well-matched markets.

### Win conditions (what a strong answer must demonstrate)
The answer needs to demonstrate all four: **systems thinking, product judgment, domain insight, and AI/ML leverage.** Working hypothesis: a sharp reviewer is probing whether the answer finds the *real bottleneck* rather than just wiring up a pipeline.

---

## Techniques Plan

1. **First Principles Thinking**: strip the problem to what is actually true underneath the current solution. (Complete)
2. **Anti-Solution / "What would break this"**: pressure-test the resulting design; surface the risks a sharp reviewer will raise. (Complete)
3. **Role Playing (broker on a live call)**: walk the design through a real conversation to find where it snaps in practice. (Complete)

---

## Technique 1: First Principles Thinking

### The reframe, in one line
There is no universal "complete profile." Completeness is defined by the carriers you target, so the profile and the carrier shortlist must be built **together, in a loop**, not sequentially.

### The chain of reasoning

**1. "Sufficient" is carrier-relative, not universal.**
An underwriter is not buying documentation or peace of mind. They do two coupled things, and only two:
- **Appetite fit:** is this risk in the box of things I am willing to write at all?
- **Priceability:** can I set a premium where expected premium beats expected losses plus expenses plus margin? Insurance is a priced bet on the frequency and severity of future claims.

Safety documentation, prior claims, and every other fact matter only as evidence that moves appetite fit or the loss estimate. Both are **carrier-specific**: Carrier A's "sufficient" is a different fact set than Carrier B's. So a context-free "complete profile" is not a coherent goal.

**2. "Collect complete, then select carriers" is the root cause, not a neutral step.**
To collect a "complete" profile *before* choosing carriers, you are forced to collect the **union of every plausible carrier's questions**. That union is exactly what the current pipeline builds: gather every carrier's question set, dedupe, chain into one giant tree. This directly generates two stated failure modes:
- The tree is huge and slow to maintain *because* it is the union.
- The call risks running long and the tree cannot cover every branch *because* it is trying to satisfy every carrier at once instead of the 2-3 you will actually submit to.

**3. The right shape is a convergent loop, not a line.**
- Collect a small, near-universal **core** that lets you triage almost any business into a first carrier shortlist.
- The carrier-side universal core is roughly four facts: **industry / class of business, size (a rating basis: revenue, payroll, headcount, or square footage), geography (licensing + catastrophe exposure), and loss history (prior claims).**
  - Note: "ability to pay premium" is a real filter, but it is **broker-side deal qualification**, not carrier appetite. Keep the two axes separate.
- That core forms an early candidate set of carriers.
- Every new answer **reshapes** the set: it can shrink it (a carrier drops out), swap it (a specialty market gets added), or unlock **new mandatory questions** that were irrelevant a moment ago.
  - Worked example: a restaurant, five carriers on the list, then "we deep-fry heavily and have had two grease fires." Some carriers drop, a specialty higher-hazard market may be added, and new questions become mandatory (hood/duct cleaning frequency, suppression system type, corrective action taken).

**4. Replace the static decision tree with a dynamic "next best question" engine.**
- **State** = partial profile collected so far + the current live carrier shortlist.
- **Step** = a policy picks the single highest-value next question given that state. "Highest value" = the question that best **prunes** the candidate carriers or **sharpens the price** for the likely winners.
- **Update** = the answer reshapes the state.
- **Stop** = the shortlist has converged to the right 2-3 *and* you have collected exactly what those three need to quote.
- This makes "sufficient" a **computable stopping condition** instead of a vibe. In ML terms this is closer to active learning / an adaptive questionnaire than a hardcoded flowchart.

**5. The LLM's job flips.**
Not "transcribe the call against a tree," but:
- Turn free-form conversation into structured risk facts, live.
- Each turn, compute the next most valuable question given who is still in appetite.

### How this dissolves the failure modes
- **No giant per-risk tree to hand-build:** the next branch is computed locally, on the fly, never enumerated in advance.
- **Messy businesses handled better:** the loop reacts to whatever fact just appeared instead of needing a pre-drawn slot for it.
- **Drop-off drops:** the call collects exactly what the live targets need, so "sufficient in one pass" becomes the default rather than the hope.

### The load-bearing risk (bridge to Technique 2)
The manual work did not vanish. It **moved**: from "maintain decision trees per risk type" to "represent each carrier's appetite and pricing sensitivities as models a machine can query at every turn." The whole approach is only as good as those carrier representations. Whether that is a better trade than the tree is the entire bet, and it needs to be stress-tested.

---

## Technique 2: "What would break this" (Anti-Solution)

Adversarial pass aimed at killing the Technique-1 loop before a reviewer does. Two attack surfaces landed hard: the carrier appetite/pricing representation, and the human reality of the call.

### Attack surface A: the carrier appetite/pricing model

**What a carrier appetite document actually is:** marketing collateral written by the carrier's distribution team to attract broker submissions. Hedge language ("we consider," "preferred," "case-by-case"). Lists rough target classes, size bands, geographies. Almost never contains the real underwriting rules, which live in internal manuals and the underwriter's judgment.

- **[Break #1] Appetite docs are marketing, not rules.** Too vague to drive precise pruning. Feeding them to the next-best-question engine means optimizing against a fiction.
- **[Break #2] Static appetite/pricing models rot.** Appetite drifts quarterly with loss experience, reinsurance costs, and catastrophe exposure. The maintenance burden the loop was meant to kill returns, relocated onto the carrier model.
- **[Break #3] Stated vs revealed appetite gap.** The system can confidently prune the carrier that would have bound the deal, because it trusts what carriers *say* over what they *do*.

**[Break #4 → turned into a strength] Learn appetite from the brokerage's own quote/bind/decline history, not from appetite docs.** Every submission the brokerage sends produces a labeled outcome (this carrier + this risk → quoted at price / declined / bound). That is *revealed* appetite: proprietary, compounding, unavailable to any competitor or PDF. It becomes a **data flywheel** that sharpens with every deal and hits all four win conditions at once.
- Caveat (the likely counter): **cold-start problem.** Young books, new carriers, and rare risk types have little history, so the model is weakest exactly where you need it most.

### Attack surface B: the human reality of the call

- **[Break #5] For document-backed facts (loss runs, payroll by class, dec pages, financials), one-pass is physically impossible.** The owner does not *know* these; they must *retrieve* them. Better question-sequencing cannot extract data that is not in the room.
- **[Break #6] Drop-off is not caused by follow-up. Drop-off = friction > motivation, modulated by starting intent.** The owner vanishes when the hassle of producing docs outweighs their motivation; they return when motivation outweighs hassle. A broker-convinced fence-sitter and an owner who already knows they need coverage are different funnel risks. Client intent should itself modulate how hard the system pushes for one-call completion (another candidate state variable for the loop).

**[Break #7 → the big reframe] Split facts into two lanes; deliver a provisional quote in-call, before the homework.**
- **Lane 1 (conversational facts):** extracted live by the Technique-1 loop, enough to produce a *provisional quote*.
- **Lane 2 (document-backed facts):** collected async with near-zero friction, e.g. broker orders loss runs directly from the prior carrier under authorization, integrates to payroll/accounting to auto-pull payroll-by-class, accepts a phone photo of a dec page, one-tap upload.
- **The provisional quote is the motivation anchor** that makes the async follow-up survive instead of drop off. The call ends with a win the client wants (a real price + a vivid picture of their current exposure), not a chore they resent.
- This **dissolves the "must land in one pass" premise** that the original problem treated as a law of nature. You don't cram everything into one call; you route facts into two lanes and put the value up front.

### Two levers against drop-off (design principles)
1. **Crush the friction of the follow-up** (auto-pull loss runs and payroll, photo-a-dec-page, one-tap async upload).
2. **Anchor motivation across the gap** (fast provisional price + concrete exposure, delivered before the paperwork).

---

## Technique 3: Role Playing (broker on a live call)

Scenario: *Sal's Trattoria*, ~15 employees, fryer + wood oven, moderate-intent owner shopping because his renewal jumped 8k to 11k, impatient, ~20 minutes before lunch rush. Walked one live call turn by turn.

- **[RP #1] System output must be a fact-target, not a script.** The broker never reads the machine's question aloud; he wraps it in acknowledgement, reassurance, and framing. Separate the **policy layer** (what fact to obtain next) from the **delivery layer** (how to ask it). Reciting raw questions makes the broker sound like a bot and rapport dies.
- **[RP #2] Over-promise guard.** The broker committed to "beating 11k" before scoping anything. The system should flag: no price commitments before core facts + loss history are in.
- **[RP #3] The client's stated price ceiling becomes a hard filter / state variable** in the loop ("if I can't beat 11k, I lose him").
- **[RP #4] Provisional quote fires from a revenue *range* + class via the flywheel,** before the firm revenue figure arrives as Lane-2 homework. The imprecise human answer is enough to anchor value.
- **[RP #5, the guardrail] Live materiality / compliance check.** When the broker is about to omit a material fact (Sal's off-the-books nephew) or over-promise, the system flags it. A working person, cash or not, is material to workers-comp and liability rating; omitting them is material misrepresentation. Protects the client (claim denial / policy rescission), the brokerage (E&O liability), and carrier trust (the core constraint of the whole problem). This is a top AI-leverage story: a regulated-industry guardrail only a real-time system can provide.
- **[RP #6] The guardrail arms the broker with a vivid, client-specific exposure scenario, not a rules lecture.** "If your nephew slices his hand and we didn't declare him, your claim is denied and you pay out of pocket." Reframes compliance as *protecting the client*; the broker becomes protector, not enforcer. The same vivid-exposure engine serves three jobs at once: motivation anchor (beat follow-up friction), compliance (secure honest disclosure), and submission story (risk understood and mitigated).

**Meta-finding:** the human never cooperates with the tree. Sal rambled, jumped to price, gave fuzzy numbers (revenue as a range), hid a worker, and watched the clock. Live proof that a recomputing loop + a delivery layer beats a rigid script, and that the system must handle **uncertainty and confidence**, not demand false precision.

---

## Emerging Candidate Direction (provisional)

A two-lane, carrier-aware capture flow with value delivered up front:

1. **Bootstrap** a carrier shortlist from a thin universal core (industry/class, size, geography, loss history).
2. **Drive the live call** with a "next best question" policy scored against the live shortlist, using only Lane-1 conversational facts.
3. **End the call with a provisional quote** from the 2-3 best-matched carriers (the motivation anchor), plus a vivid statement of current exposure.
4. **Collect Lane-2 document-backed facts async** with near-zero friction to firm up the quote and finalize the submission.
5. **Package** the collected facts into a per-carrier submission story.

Center of gravity / main thing to design and defend: a **queryable carrier appetite + pricing model learned from the brokerage's own bind/decline history** (a data flywheel), not scraped from appetite PDFs. The cold-start weakness is the known risk to address.

---

## Open Threads / Next Steps
- [x] **Technique 1 (First Principles):** complete. Loop architecture + next-best-question engine.
- [x] **Technique 2 (Anti-Solution):** complete. Carrier model = data flywheel from own history; two-lane fact routing + provisional-quote anchor.
- [x] **Technique 3 (Role Playing):** complete. Policy-vs-delivery split, over-promise guard, price-ceiling state var, provisional-from-range, and the live materiality/compliance guardrail armed with vivid client-specific exposure.
- [x] **Organize + prioritize** into a short set of candidate approaches with rough next steps: done. See the Synthesis sections, the Hardened Synthesis (scorecard, sequencing, risks, demo), and the standalone design doc (`system-design.md`).

---

## Synthesis: Themes

The 19 findings cluster into four themes.

- **Theme 1: The capture engine is a loop, not a tree.** (FP1-5, RP1 meta) A dynamic next-best-question policy over a live carrier shortlist; "sufficient" becomes a computable stopping condition; the LLM structures speech into facts *and* selects the next fact to obtain.
- **Theme 2: Carrier matching is a data problem, and the data is proprietary.** (FP6, Break1-4, RP3) The moat is *revealed* appetite learned from the brokerage's own quote/bind/decline outcomes, not appetite PDFs. Cold-start is the risk. The client's price ceiling is a filter input.
- **Theme 3: Kill the drop-off by reframing "one call," not by cramming it.** (Break5-7, RP4) Two-lane fact routing; a provisional quote up front as the motivation anchor; frictionless async collection of document-backed facts; client intent modulates how hard to push.
- **Theme 4: The broker is the interface, and trust is the product.** (RP1-2, RP5-6) The copilot outputs fact-targets, not scripts; guardrails against over-promising and material misrepresentation; vivid client-specific exposure narratives that serve motivation, compliance, and the submission story at once. E&O and carrier-trust protection.

---

## Synthesis: A Short Set of Candidate Approaches

These are four distinct bets. They compose into one system, but each is a defensible scoping choice on its own, and the strength is in how you *sequence* them.

### Approach 1: Adaptive Capture Loop
Replace the static per-risk decision tree with a next-best-question *policy* over a live carrier shortlist. State = partial profile + live shortlist; each turn, ask the fact that best prunes carriers or sharpens price; stop when the shortlist converges to 2-3 and you have what they need.
- **Serves:** systems thinking, AI/ML. Kills tree maintenance; handles messy businesses.
- **Risk:** only as smart as the carrier model behind it (depends on Approach 2).

### Approach 2: Revealed-Appetite Flywheel
Learn which carriers actually bind which risks from the brokerage's own outcome history. Every submission logs (carrier + risk profile → quoted at price / declined / bound). That proprietary, compounding dataset drives the "right 2-3 carriers" selection.
- **Serves:** the core constraint directly (target the right markets, do not spray, protect carrier relationships). Strongest domain + moat story.
- **Risk:** cold-start on young books, new carriers, rare classes.

### Approach 3: Provisional-Quote-First, Two-Lane Funnel
Reframe the problem from "everything in one call" to "value up front + frictionless async." Deliver a provisional quote in-call from the easy conversational facts, then collect document-backed facts async with near-zero friction (auto-order loss runs under authorization, integrate payroll/accounting, photo a dec page).
- **Serves:** product judgment. Attacks the actual business metric (drop-off) and dissolves the premise the prompt treats as law.
- **Risk:** async-collection integrations; provisional-quote accuracy vs over-promising.

### Approach 4: Broker Copilot with Compliance Guardrail
A real-time copilot that (a) transcribes and structures the call into a fact schema with confidence, (b) whispers fact-targets with a delivery layer (never a script to recite), (c) guards against over-promising and material misrepresentation, and (d) arms the broker with vivid, client-specific exposure narratives.
- **Serves:** AI-leverage-in-a-regulated-industry, the most defensible "why AI here." Protects client from claim denial, the brokerage from E&O, and carrier trust.
- **Risk:** latency during a live call; broker adoption; LLM mis-structuring compounding through the loop (needs confidence + human confirmation).

---

## Synthesis: Prioritization and the Through-Line

The four compose into one system, but you cannot build them in parallel, because Approach 2 (the moat) has a cold-start problem: you have no outcome data on day one. So the sequencing *is* the strategy, and it is the sharpest single thing to say:

**The MVP must be designed to generate the training data for the moat. The copilot's logging IS the flywheel's fuel.**

- **Phase 0 / MVP (ships now, no cold-start):** Approach 4 + Approach 3. A broker copilot that structures the call, routes facts into two lanes, runs the compliance/over-promise guardrail, and produces a provisional quote from *whatever* carrier signal you can bootstrap (appetite docs + heuristics are fine here). Delivers value on day one and, critically, logs every submission outcome.
- **Phase 1 (the moat forms):** Approach 2. Those logged outcomes become the revealed-appetite dataset. Swap heuristic carrier matching for the learned flywheel. "Right 2-3 carriers" gets sharp; the moat compounds.
- **Phase 2 (the loop gets optimal):** Approach 1. With real appetite/pricing signal, the next-best-question policy becomes genuinely information-optimal, and "sufficient" becomes a computable stopping condition.

This ordering shows you understand you cannot build the moat directly; you have to *earn* it, and the product's daily use is what earns it.

### Risks to pre-empt (name these before a reviewer does)
1. **Cold-start** on the appetite model (mitigation: bootstrap with heuristics; the MVP's job is to accumulate data).
2. **LLM mis-structuring / hallucination compounding through the loop** (mitigation: confidence scores, human confirmation on low-confidence or high-impact facts, never silently prune on a shaky fact).
3. **Provisional-quote accuracy vs over-promising** (mitigation: ranges with explicit caveats; the over-promise guardrail).
4. **Regulatory / E&O exposure** (mitigation: the materiality guardrail is a feature, not an afterthought).
5. **Broker adoption / latency** on a live call (mitigation: fact-targets not scripts, sub-second suggestions, broker always in control).

---

## Rough Next Steps (for building the response)
1. **Pick one risk class** (e.g. restaurants) and take it end to end, so everything is concrete.
2. **Define the fact schema:** universal core (industry/class, size, geography, loss history) + class-specific facts, each tagged **Lane 1 (conversational)** or **Lane 2 (document-backed)**.
3. **Define the outcome-log schema** (the flywheel fuel): carrier + risk profile snapshot + result (quoted/price/declined/bound). Design this on day one even though the model comes later.
4. **Sketch carrier matching:** v0 heuristic from appetite docs; v1 learned from the outcome log. Be explicit about the transition.
5. **Design the copilot loop:** live transcription → structured facts with confidence → next-best-fact target → delivery prompt; plus guardrail rules (materiality checklist, over-promise triggers) and the vivid-exposure generator.
6. **Design the two-lane funnel:** provisional-quote mechanism + frictionless async collection (auto-order loss runs, payroll integration, photo/upload).
7. **Name your metrics:** one-call sufficiency rate, drop-off / come-back rate, quotes-per-submission (carrier hit rate), bind rate, and submission-to-bind ratio per carrier (carrier-relationship health, the core constraint made measurable).
8. **Write the risks section** with the mitigations above; leading with your own risks reads as senior and self-aware to any reviewer.

> Note: everything below supersedes the inline synthesis above. The inline version is the pre-hardening view; the Hardened Synthesis is the authoritative recommendation after adversarial review.

---

# Hardened Synthesis (multi-agent adversarial pass)

Produced by 5 independent rival designers (underwriter, ML-platform, growth, ruthless-MVP, contrarian), 5 adversarial red-teamers, and 2 blind-spot critics, then a high-effort synthesizer. This is the version to actually walk into the room with.

## What changed, and why

The pass landed **two fatal hits** on the original recommendation and forced real changes:

1. **The volume moat is dead at startup scale (fatal).** At realistic volume (~5k submissions/year firm-wide, power-law across classes, concentrated on 2-4 carriers per class), the tail cells that justify the product see ~2-27 submissions/carrier/year. Layer on: you only submit to 2-3 carriers so 9+ carriers are *censored/never observed*; a trustworthy "good outcome" label (bound-at-good-price-that-renewed-clean) takes 18-30 months to mature; and appetite is non-stationary with a 6-12 month half-life. **In the cells that matter, the signal decays faster than the cell accumulates events.** Worse, the next-best-question loop trains only on carriers the bootstrap heuristic already shortlisted, so it learns to *confirm its own priors*, and a single LLM extraction error logs a confident false decline that poisons the flywheel.
   - **Fix:** rebuild the flywheel as a **data-EFFICIENCY** asset, not a volume one. A hierarchical / low-rank Bayesian model where **the appetite doc IS the prior** and your logged outcomes are the residual; partial pooling across correlated carrier-risk cells (useful signal at 20 events, not 20,000); graceful degradation to the prior when a cell is empty; a bounded **exploration budget** (Thompson sampling) so the loop doesn't just confirm itself; and calibrated-confidence extraction that is **forbidden from pruning a carrier on a low-confidence fact** (it asks the disambiguating question instead).

2. **The revealed-appetite objective is an adverse-selection engine (fatal).** Optimizing "lowest price that binds" mathematically routes each carrier exactly the risks that slip past its guidelines but bind cheap, i.e. the accounts its manual was trying to keep out. Your book with that carrier develops a structurally worse loss ratio; carriers grade producers on exactly this; the damage is invisible for 18 months, then they re-price your book, cut commission, or pull the appointment. **The flywheel silently torches the very carrier relationships the core constraint exists to protect.**
   - **Fix:** flip the objective from lowest-price-that-binds to **predicted pull-through + persistency + clean loss**. Enforce a **submission-discipline budget** (cap submissions per carrier, only send high-fit). Make it **carrier-cooperative**: bring cleaner, better-fit, higher-persistency submissions in exchange for structured appetite feedback and preferred-market status. Sell the carrier a better book, not a mispriced one.

Three more forcing findings:

3. **Packaging is the actual product, not a schema-to-PDF step.** A "sufficient," correctly-matched, cleanly-rendered submission still gets declined if the loss history is presented as raw facts instead of a *remediated story* (two slip-and-falls in 2022 → then installed non-slip flooring → 24 months clean; a good broker leads with the remediation). And that failure is **invisible to the flywheel**: it logs "carrier declined this profile" and learns the *wrong* lesson, steering future well-remediated risks *away* from a carrier that would have bound them.
   - **Fix:** elevate a **submission-quality / narrative engine** to first-class. **Log decline reasons** to split appetite-mismatch from story-failure. **A/B narrative framings** on the same profile to make packaging a learned, compounding, un-scrapeable asset (deeper moat than appetite matching). Guardrail the persuasion: foreground mitigants, never soften the claim itself.

4. **Compliance is an architectural layer, not a guardrail feature.** The in-call provisional quote is three violations in one moment: **unlicensed quoting** (software generating a premium indication), **detrimental reliance** (the owner relies on a number that lands 40% higher post loss-run → E&O), and **misrepresentation knowledge** (the guardrail catching a hidden worker creates written proof the brokerage *knew*). Every transcript becomes discovery against the brokerage.
   - **Fix:** reclassify the quote as a **non-binding indicative range with a state-appropriate disclaimer, attributed to a named licensed producer of record** (AI proposes, the human asserts, the same policy-vs-delivery split applied to legal actorhood). Make **state-of-risk an early mandatory pruning fact** and hard-branch licensing / surplus-lines logic. The guardrail becomes a **mandatory stop**: on detected material misrepresentation, the loop cannot proceed without documented cure-or-decline. Reframe: because insurance is adversarially regulated, the winning product is the one whose every step emits a timestamped standard-of-care record. **Compliance logging is the same asset as appetite logging: the moat's substrate AND the best market-conduct-exam / E&O defense file a brokerage has ever had.**

5. **The copilot must go near-silent during the call.** A good broker is already at ~100% cognitive load (reading tone, building rapport, deciding when to shut up). A live copilot emitting a shifting fact schema, carrier shortlist, compliance warnings, and a quote engine adds a *second live conversation* and makes the best brokers worse (they glance down, miss "we had a slip-and-fall last spring but it was nothing"). And per-minute live inference against a few-hundred-dollar commission at 20-40% bind is negative unit economics.
   - **Fix:** near-silent during the call (passive structuring + at most 1-2 ambient, non-blocking nudges); do the real reasoning in a **60-second post-call pass**. Cheap-model the transcription, reserve expensive inference for post-call packaging, gate quote compute to qualified prospects. **Re-anchor ROI on submission assembly and re-contact chase (where hours and drop-off actually die), not on faster talking.** Pilot with mid-tier brokers who gain the most and resist least; earn top-producer trust with a bind-rate scoreboard, not a confidence score.

## Strongest additions from the rival designers

- **Classification-first underwriting (the single best add).** Before the call, resolve ISO/NAICS class code + rating basis + hazard grade from name, address, and website + third-party data. This **eliminates 60-70% of tree questions by construction** (a class code implies most answers) and turns messy edge cases into a *story field*, not a missing branch.
- **Appetite as three explicit sets per carrier**, not a question list: **knockouts** (declines that kill a submission), **rating-drivers** (the 5-10 fields that actually move price/acceptance), **story-boosters** (differentiators underwriters reward).
- **Sufficiency and carrier-match are the same model queried two ways**, computed *jointly and live*. "Sufficient" = the top-3 set is stable AND no hard-eligibility field for those 3 is still unknown.
- **A live sufficiency meter** as a visible finish line and commitment moment ("we're taking this to these two carriers today") that closes the loop before drop-off; plus instrument the *full* funnel including booking-to-show and submission-to-carrier-engaged leaks the tree never touches.
- **The contrarian "buy the map, don't grow it" correction** for the near term: the broker names the target markets and the system does live **gap-detection against a curated requirements library**; a conservative, explainable, human-confirmed shortlist protects carrier relationships better than a probabilistic matcher.

## Scorecard (1-5, after hardening)

| Approach | Systems | Product | Domain | AI/ML | Feasibility | Verdict |
|---|---|---|---|---|---|---|
| A: Adaptive capture loop (VOI) | 5 | 4 | 3 | 4 | 3 | Survives only with an exploration budget + never-prune-on-low-confidence rule |
| B: Revealed-appetite flywheel | 4 | 3 | 2 | 3 | 2 | Two fatal hits; survives only rebuilt as hierarchical-Bayes efficiency model optimizing pull-through |
| C: Provisional-quote / two-lane | 4 | 5 | 4 | 3 | 3 | Best product insight; survives only as disclaimed indicative range + LOA-gated Lane 2 |
| D: Broker copilot + guardrail | 4 | 4 | 4 | 4 | 3 | Right instincts; survives near-silent-during-call + 60s post-call + mandatory-stop guardrail |
| Integrated Phase 0→1→2 | 5 | 4 | 4 | 4 | 3 | Sequencing survives only if the moat is redefined as efficiency + packaging + carrier-trust |

## Rewritten sequencing (the through-line to say out loud)

The original "the MVP earns the volume moat" is **false at the brokerage's scale** and was the most load-bearing wrong assumption. Keep the sequencing, change what each phase earns:

- **Phase 0 (ship now):** classification-first pre-call enrichment + a curated LLM-parsed requirements/appetite library (knockouts + rating-drivers + story-boosters) + live gap-detection against broker-named target markets + a joint sufficiency/match meter + an auto-drafted **carrier-specific narrative**, all logging **calibrated-confidence facts AND decline reasons.** The Phase-0 moat is **packaging craft + workflow lock-in + a defensible, explainable RIGHT-3 decision**, available day one, not a promise of future data.
- **Phase 1 (gated, not scheduled):** turn accumulated outcomes into the hierarchical-Bayesian appetite-*efficiency* model (prior = appetite doc) optimizing pull-through/persistency, activated **segment-by-segment only when a pre-stated significance threshold is hit**, with a bounded exploration budget.
- **Phase 2 (VOI loop):** only drives broker prompts *after* it beats the rules+library baseline in shadow mode on held-out bind/persistency.

Honest one-liner: **year one's defensible asset is a calibrated extraction-plus-packaging engine that is more data-efficient per submission than anyone relying on stated appetite; the volume/network moat is a gated bet layered on top, not the launch thesis.**

## Blind spots to fold in

- **Placement graph is multi-tier and multi-axis:** the object is a "market" (direct carrier / wholesaler / MGA / program) on an admitted-vs-E&S axis. First decision is often admitted vs non-admitted; E&S carries diligent-search / declination-documentation / surplus-lines-tax obligations the one-pass framing ignores. The flywheel is bounded by the brokerage's actual appointment footprint.
- **Accounts are multi-line, not mono-line.** Carriers underwrite and cross-subsidize the whole account (write the WC to get the property), so the shortlist is a bundling/splitting decision and some facts prune carriers across multiple lines at once.
- **Segment where narrative actually matters.** Commoditized / portal-bound small commercial is rate-driven off a few knockout + rating variables, so there "compelling" = clean, eligibility-passing structured data to the right rater; narrative earns its keep only in human-underwritten mid-market / E&S. Add **eligibility-knockout pre-checks** as a first-class, cheap, high-signal capability.
- **Renewals, not just new business.** Renewals are 85-95% retention, the real LTV driver, and a far cleaner, warmer, delta-based feedback signal for the flywheel (mitigates cold-start faster) than cold new business.
- **The contingent-commission conflict.** Brokers are paid more on higher premium and by the carrier, which collides with a client-best-fit objective. Surface it as a documented, possibly disclosed, secondary factor and an E&O/fiduciary point.
- **Competitive read.** Newfront, CoverWallet, Embroker, Vouch, wholesalers-with-tech, and carriers' own underwriting AI. The brokerage's outcome data is proprietary but SMALL vs incumbents, so the wedge is broker-in-the-loop on harder/non-standard risk + packaging craft, not raw data volume.

## Surviving risks (name these before a reviewer does)

| Risk | Severity | Mitigation |
|---|---|---|
| Flywheel never reaches significance in tail cells before appetite drifts | serious | Data-efficiency reframe (Bayesian prior = appetite doc), partial pooling, gate Phase 1 on a significance threshold, seed with bought/partnered priors |
| Adverse selection torches the producer scorecard on an 18-month lag | serious | Optimize pull-through/persistency/clean-loss; submission-discipline budget; carrier-cooperative feedback loop |
| In-call quote = unlicensed quoting + detrimental reliance + misrep knowledge | serious | Disclaimed indicative range attributed to a licensed producer of record; state-of-risk early; guardrail as mandatory cure-or-decline stop |
| Packaging failure is invisible to and poisons the flywheel | serious | First-class narrative engine; log decline reasons; A/B framings; guardrail advocacy vs misrepresentation |
| Copilot adds live load, hurts best brokers, negative unit economics | serious | Near-silent during call; 60s post-call pass; cheap transcription, gated quote compute; ROI on assembly + re-contact |
| Mono-line / direct-carrier model misses multi-line + wholesalers + E&S | manageable | Account-level state; shortlist = "markets"; admitted-vs-E&S first; surplus-lines as Lane 2 |
| Sufficiency mis-defined vs a stale requirement schema → the callback returns | manageable | Define sufficiency against carrier required-to-bind field sets incl. subjectivity patterns; minimize expected callbacks weighted by drop-off cost; async Lane 2 as graceful catch |

## Recommended demo scope (what to actually build)

**One vertical, end to end: restaurants/hospitality, GL + property (a real BOP-shaped account).** Two artifacts, no decision tree:
1. A flat ~40-fact **risk schema**.
2. **4-6 real market appetite profiles** as declarative **knockouts + rating-drivers + story-boosters**, authored LLM-assisted from an appetite doc + a human edit pass (show this as the maintenance artifact).

- **Pre-call:** resolve class code + hazard grade from name/address/website.
- **Live:** LLM open-extraction from a sample transcript into the schema with **per-fact confidence**; after each utterance a deterministic scorer marks each market eliminated/qualified/needs-info, and a **sufficiency meter** names the top-3 and surfaces the single highest-VOI next question, **refusing to eliminate a market on a low-confidence fact.**
- **Finish:** meter says "stop, you can win these three" and **auto-drafts one carrier-specific submission narrative** that foregrounds a remediated loss (the two-slip-and-falls-then-non-slip-flooring case), proving packaging, not form-filling.
- **One visible compliance moment:** the indicative range renders with a state disclaimer + producer-of-record attribution, and a detected off-the-books worker triggers a mandatory cure-or-decline stop.
- **Deliberately NOT built:** multi-risk taxonomy, telephony/ASR, CRM plumbing, the learned flywheel (show it as an instrumented logging stub + the napkin-math significance argument), carrier portal.

## The six sharpest things to say

1. "A 100%-complete profile can be unbindable because it trips one knockout no tree flagged, and a 40%-complete profile binds instantly because the class is clean and in-appetite. So we compute sufficiency and carrier-match jointly and live: they're the same model queried two ways, not two hand-built artifacts."
2. "We're not reverse-engineering your underwriters' appetite to win the cheapest quote; that's an adverse-selection machine that torches our producer scorecard on an 18-month lag. We optimize predicted pull-through and persistency, so we raise your hit ratio and lower your noise. We sell you a better book."
3. "At our volume a per-cell frequentist appetite model is dead on arrival, and I can show you the napkin math. So year one's moat is data-efficiency: appetite docs are our Bayesian prior, our outcomes are the residual, and we run a bounded exploration budget so the loop never just confirms its own priors."
4. "Our compliance layer isn't a tax on one-call speed; it's the asset. Insurance is adversarially regulated, so every step emits a timestamped standard-of-care record. The same logging that fuels the model is the best market-conduct-exam and E&O defense file any brokerage has ever had."
5. "Underwriters don't decline because a field was blank; they decline because nobody gave them a reason to say yes and a way to defend the account internally. So packaging is the product: we A/B the narrative framing, make how-you-tell-the-story a learned, un-scrapeable moat, and log decline reasons so the model never confuses a bad write-up with no appetite."
6. "The copilot is near-silent during the call; it captures passively and reasons in a 60-second post-call pass. The scarce broker resource isn't call minutes, it's submission assembly and re-contact chase, and that's where the drop-off and the hours actually die."
