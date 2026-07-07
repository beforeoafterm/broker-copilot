# How This Got Built

### Turning a vague conversation into a solution that survives being attacked

**The gist, in ten seconds:** Nobody assigned this. It came out of a loose chat, so I treated it like real requirements: fuzzy, under-specified, and mine to figure out. I used AI not to hand me an answer, but to think in disciplined passes and then to try very hard to prove itself wrong. It found two fatal flaws in its own first solution. Fixing those is the whole story.

---

## Why this exists

No brief was handed to me. The problem surfaced in a meeting, described loosely, the way requirements actually show up in real life. So I built this on my own to demonstrate one thing: give me a vague problem in a domain I don't know, and I will get to a defensible answer, with AI as a genuine partner rather than a vending machine.

The solution itself lives in [system-design.md](system-design.md). This page is about how it got made, because the method travels and the solution is just one thing it happened to produce.

Here is the honest version: a human made every judgment call and owns the reasoning. AI brought structure, insurance facts on request, and, most importantly, the ability to attack the answer from more angles than one person and one chat window ever could. The first tidy answer was not allowed to win. It had to earn it by surviving a beating.

---

## The pipeline

```
  VAGUE CONVERSATION
        │
        ▼
  ┌─────────────┐   Pin down what a good answer must prove.
  │  1. FRAME   │   Turn "we talked about a thing" into a scorable target.
  └─────────────┘
        │
        ▼
  ┌─────────────┐   Structured brainstorming, 3 techniques, asked as
  │ 2. DIVERGE  │   questions, not lectures. 19 raw ideas out.
  └─────────────┘
        │
        ▼
  ┌─────────────┐   Cluster the mess, pick the few real candidates,
  │ 3. CONVERGE │   commit to a plan. Feel briefly clever.
  └─────────────┘
        │
        ▼
  ┌─────────────┐   Hand the plan to 13 AI agents whose only job is
  │ 4. HARDEN   │   to embarrass it. Stop feeling clever.
  └─────────────┘
        │
        ▼
  ┌─────────────┐   A design doc and a buildable demo, now with the
  │ 5. DELIVER  │   holes already found and filled.
  └─────────────┘
```

Two ideas do all the heavy lifting: spread out before you narrow down (so your first idea doesn't quietly rig the vote), and then attack your own answer on purpose, before anyone else gets the chance.

---

## 1. Frame

Before generating anything, I wrote down what a strong answer has to prove: systems thinking, product judgment, real insurance insight, and genuine AI leverage. Naming the bar up front turns a vague ask into something you can actually score yourself against. It also keeps you honest later, when the fun ideas start arriving and you want to skip the boring test of "but does this matter."

## 2. Diverge

Structured brainstorming, three techniques, each picked for a different job. Run as facilitation, not dictation: the AI asked me sharp questions and only fed me insurance facts when I asked for them, so the thinking stayed mine.

- **First principles** stripped the problem to bedrock and cracked it open: "sufficient" is defined by each carrier, so there is no such thing as one complete profile, and collecting the data and picking the carriers are the same job, not two steps.
- **"What would break this"** went looking for where the shiny idea dies. It found that carrier appetite documents are marketing brochures, not rulebooks, and that the real enemy is not the follow-up, it is friction beating motivation.
- **A role-played sales call** put the design in a room with a real, impatient restaurant owner who was cheerfully trying to hide his off-the-books nephew. That one call surfaced the compliance guardrail, the "don't make the broker sound like a robot" rule, and more.

Nineteen findings came out. Three lenses, because each catches what the others miss.

## 3. Converge

I clustered the nineteen findings into four themes, boiled those into four candidate approaches, and lined them up into one phased plan. This is where most processes stop, holding a confident, well-argued recommendation.

That confidence was the trap.

## 4. Harden (the fun part)

I handed the tidy recommendation to thirteen AI agents whose entire purpose was to make it look foolish:

- **Five rival designers**, each with a different personality (a grizzled underwriter, an ML platform engineer, a growth lead, a ruthless MVP-first founder, and a contrarian told to argue the whole thing was over-engineered). They designed independently, without seeing my answer, so they had no reason to be polite about it.
- **Five red-teamers**, each attacking from exactly one angle: regulation and liability, carrier politics, the actual machine-learning math, unit economics, and the half of the problem I had quietly under-cooked.
- **Two critics** hunting for entire topics I never even brought up.

Then one more agent to referee and rebuild.

They were rude. They were also right. Two of my four ideas had fatal flaws:

1. My "the product builds a data moat" story. The math does not survive a startup's actual volume. Dead on arrival.
2. My carrier-matching logic. It was quietly an adverse-selection machine that would slowly poison the exact carrier relationships the problem told me to protect. Which is a bit like designing a smoke alarm that starts fires.

Both got rebuilt. Three other ideas got sharper. And the single best new idea in the whole solution came from the agent pretending to be a crusty underwriter.

The point is not that my first answer had holes. Every first answer has holes. The point is that the process is built to find them before anyone reviewing it does.

## 5. Deliver

Out came two things: a design document balanced across all four success criteria, and a deliberately small demo (one restaurant, done properly) that proves the core idea without trying to build the entire company. Both lead with their own weak spots on purpose. Owning a flaw reads as senior. Getting caught hiding one reads as the opposite.

---

## What this actually demonstrates

- I can turn "we discussed something in a meeting" into a real architecture, in an industry I am new to.
- I use AI the way an AI-native company should want it used: for structure, for expertise on demand, and as a built-in red team, with a human holding the wheel.
- I treat my own first answer as a suspect, not a verdict.
- None of this is specific to any one company. The same five steps run on any vague ask you throw at them.

## The fine print (because pretending otherwise would be worse)

- The AI's insurance knowledge is scaffolding, not scripture. Every domain claim here is a hypothesis to check with a real underwriter.
- The numbers that killed my two bad ideas (submission volumes, how fast appetite drifts) are educated estimates, and the design flags them plainly as things to confirm against your real data.
- I made the calls. AI just made me roughly ten times faster at exploring them and far more ruthless at testing them.

The solution is one output. The method is the asset.
