# Winning the One-Call Submission

### Capture a sufficient risk profile in a single call, and package it for the right 2-3 carriers.

**The 30-second version:** The current approach tries to ask every carrier's questions on one call and pick carriers afterward. That is backwards, and it is why the call is long, the tree is unmaintainable, and prospects vanish. Flip it: figure out which handful of carriers this business can win *while you are still on the phone*, ask only what those carriers actually care about, and end the call with a price and a story instead of homework. The hard part is not asking questions. It is knowing, live, when you have enough to win the right markets, and then telling the story that makes an underwriter say yes.

---

## The one idea everything hangs on

An underwriter decides two things, and only two: is this risk in my appetite at all, and can I price it to make money. Both are specific to each carrier. Carrier A and Carrier B care about completely different facts about the same restaurant.

So "a complete profile" is a mirage. There is no such thing in the abstract. **A profile is only complete relative to the carriers you are submitting to.** Which means capturing the profile and choosing the carriers are not two steps. They are the same step, and it has to happen live, together, on the call.

Everything below is just taking that seriously.

---

## What we'd build

A live "sufficiency and match" engine that sits with the broker on the call and does three things:

1. **Figures out the business before the call even starts.** From the name, address, and website, it classifies the risk (what kind of business, how hazardous, how it gets rated). That alone answers most of what a questionnaire would have asked.
2. **Names the right 2-3 carriers in real time, and asks only what they need.** As the owner talks, a live "sufficiency meter" shows the broker which carriers this business can win and the single most useful next question to lock one in. When the meter says "you can win these three, stop," the broker stops.
3. **Writes the winning story.** It drafts a submission tailored to each carrier that leads with the strengths and frames the weak spots, the way a great broker does, instead of dumping raw data on an underwriter's desk.

The owner hangs up with a price and a reason to come back. The paperwork that cannot be done live gets collected afterward in thirty frictionless seconds, not in a dreaded second phone call.

---

## How it works

Here is the architecture. Each piece opens with the plain-English point, then the detail for the technically inclined.

**Classify first, ask second.**
*In plain English:* most of what you need about a business follows from knowing what kind of business it is. So figure that out up front and skip the obvious questions.
Before the call, resolve the class code and hazard grade from public data (business name, address, website, prior carrier where available). This front-loads 60-70% of the profile and turns a weird, hard-to-categorize business into a "story to tell" rather than a "branch the tree forgot."

**One shared fact sheet, two lanes.**
*In plain English:* some facts the owner just knows and can say. Others (five years of loss runs, exact payroll) they have to go dig up. Treat those differently.
Every fact lives in one normalized schema, tagged Lane 1 (say it on the call) or Lane 2 (retrieve it later). Trying to force Lane 2 facts onto the live call is what was killing the funnel, because the owner physically does not have them in the room.

**Model appetite as knockouts, not questionnaires.**
*In plain English:* for each carrier, you only need to know the few things that get you rejected and the few things that move the price. Everything else is noise.
Each carrier is described by three short lists: knockouts (what kills the submission), rating drivers (the 5-10 facts that move price and acceptance), and story-boosters (what an underwriter rewards). This is dramatically lighter to build and maintain than a giant decision tree, and it is authored with AI help from the carrier's own appetite documents plus a human edit.

**Pick the next question by what it's worth.**
*In plain English:* ask the one question that best narrows down which carriers are still in play. Don't read a script.
At each moment the system scores every unknown fact by how much it would change the carrier shortlist, and surfaces the most valuable one. "Sufficient" becomes a precise, computable finish line: the top 2-3 carriers are locked and nothing they require is still unknown. And a crucial safety rule: the system never eliminates a carrier based on a fact it is unsure about. If it is unsure, it asks.

**Write the story, don't fill the form.**
*In plain English:* underwriters do not reject you for a blank field. They reject you when nobody gave them a reason to say yes.
A restaurant with two slip-and-fall claims in 2022, then new non-slip floors and two clean years, is a *good* risk if you tell it right. Presented as a naked "2 claims" line, it gets declined in eight seconds. So the system drafts a carrier-specific narrative that leads with the fix and frames the history. It also logs *why* a carrier declined, so the model can tell the difference between "wrong carrier" and "bad write-up." Without that, it would learn your worst habits and repeat them.

**Compliance is built into the walls, not bolted on.**
*In plain English:* insurance is heavily regulated, so every step has to leave a clean paper trail. That paper trail is also your best legal defense.
The in-call price is never a "quote." It is a clearly-disclaimed indicative range, attributed to a licensed human producer who is the one actually saying it (the AI suggests, the human asserts). If the owner tries to hide something material, the system stops and requires it be fixed or the submission declined. The same logging that trains the model doubles as the best market-conduct and liability defense file a brokerage has ever had.

**The copilot shuts up during the call.**
*In plain English:* a good broker on a call is already at full mental capacity building trust. A chatty AI in their ear makes them worse, not better.
So the copilot listens quietly during the call and does its real thinking in a 60-second pass right after. This also fixes the cost math: cheap models transcribe live, the expensive reasoning runs once, after, on prospects worth it.

---

## Where AI earns its keep, and where it doesn't

AI genuinely changes the game in four spots: understanding messy live speech, reasoning about sufficiency and match together, writing the carrier-specific story, and catching compliance problems in real time.

Where it would be theater: a lot of small, commodity business is priced by an algorithm off a few numbers. There, a beautiful narrative is wasted effort, and the win is clean, complete data hitting the right rater. Knowing *when* the story matters (bigger and non-standard risks) versus when it doesn't (commodity) is itself the domain judgment. We build for both.

---

## The moat, honestly

The brokerage's data is proprietary but small next to big brokers and the carriers themselves, so the durable advantage is not a giant data pile. It is four things: getting more signal per submission than anyone relying on marketing docs, learned story-craft that cannot be scraped from a PDF, a track record of sending carriers clean, well-matched business that earns capacity instead of burning it, and workflow lock-in. The wedge against digital competitors (CoverWallet, Embroker, Newfront and friends) is the harder, non-standard risks their self-serve forms cannot handle.

---

## The plan

*In plain English:* ship real value on day one, and treat the fancy learning model as a bet you earn the right to make, not a launch promise.

- **Phase 0, now:** classify-first, the appetite library, the live sufficiency meter, the story generator, and honest logging. The advantage on day one is story-craft plus a defensible, explainable "here are your 3 carriers" answer. No waiting for data.
- **Phase 1, when earned:** turn accumulated outcomes into a smarter appetite model, switched on carefully, one segment at a time, only once there is enough data to beat the simple version.
- **Phase 2:** let the model start driving the questions, but only after it has beaten the baseline in safe shadow mode.

The honest headline: year one's real asset is a system that reads a business and writes its story better than anyone relying on stated appetite. The big data moat is a bet layered on top, not the opening move.

---

## What could go wrong (and what we do about it)

| The risk | How bad | What we do |
|---|---|---|
| The learning model never gets enough data to matter | Serious | Use appetite docs as a starting prior and learn the *difference*; pool across similar carriers; only switch it on where it beats the baseline |
| Matching on "cheapest bind" quietly feeds carriers bad risks and torches the relationship | Serious | Optimize for business that *sticks and stays profitable*, cap submissions per carrier, make it cooperative with carriers |
| The in-call price creates legal and licensing exposure | Serious | Disclaimed indicative range, attributed to a licensed human; hard stop on any material misrepresentation |
| A badly-framed submission gets logged as "no appetite" and poisons the model | Serious | Log decline *reasons*; separate "wrong carrier" from "bad write-up"; A/B the story framing |
| The copilot slows brokers down and the unit cost eats the commission | Serious | Silent on the call, thinks after; cheap live transcription, expensive reasoning gated to real prospects |
| We assumed one carrier per risk, but real accounts are multi-line and often go through wholesalers | Manageable | Treat the account as the unit and the shortlist as "markets," including wholesalers and surplus lines |

Two more worth naming out loud, because a sharp reader will spot them anyway: commercial accounts are usually multi-line (a restaurant needs property, liability, workers comp, maybe liquor), and brokers get paid more on higher premium, which quietly conflicts with "best fit for the client." Both are handled in the design, and both are the kind of tension worth putting on the table early rather than leaving to be discovered later.

---

## The demo we'd build

One vertical, done end to end: restaurants, property plus liability. Not a slideware mockup, a working slice:

- A ~40-fact schema and 4-6 real carrier appetite profiles (knockouts, rating drivers, story-boosters).
- Pre-call: classify the business from its address and website.
- Live: pull facts from a sample call transcript, mark each carrier in / out / needs-info, and show the sufficiency meter naming the top 3 and the single best next question. It refuses to drop a carrier on a shaky fact.
- Finish: the meter says "stop, you can win these three," and it auto-drafts one carrier-specific submission that leads with a remediated loss.
- One visible compliance moment: the disclaimed range, and a hard stop when the owner tries to hide a worker.
- Deliberately *not* built: every other vertical, phone integration, the full learning model (shown as a logging stub plus the napkin math on why it waits).

The demo proves exactly one claim: the machine can say "stop, you can win these three, ask this one thing, and here is the story that wins" better than a decision tree ever could.

---

## How we'd know it's working

The numbers that matter: how often a single call reaches "enough to bind," how many prospects come back for the async paperwork, how many quotes we get per submission, how many bind, and, most importantly, the health of each carrier relationship (are we sending them business that sticks). Plus how much the story-framing actually lifts the bind rate.

---

## What we'd need from the brokerage

The plan's emphasis depends on facts only the brokerage has: how much of the book is commodity versus bigger non-standard risk, real submission volume, how much is renewals versus new business, and the business model. We would rather ask than assume, and saying so is part of the answer. The strongest version of this system is one honest enough to keep questioning its own premises, including whether a live phone call is even the right way to capture the easy facts, or whether the broker's rare talent should be saved for the parts only a human can do: reading the client, telling the story, and closing.
