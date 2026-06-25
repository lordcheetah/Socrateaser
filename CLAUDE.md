# Socrateaser

An agent system for working out philosophy. It takes a single arguable thesis and
runs it through a pipeline that forces genuine disagreement, checks the argument's
form, grounds claims in primary sources, and locates the exact premise where
reasonable views diverge.

## What this system is — and is not

A game studio or a book studio is a **production** system: agents divide labor
toward an artifact, and quality is craft plus taste. Philosophy is different. It
has **correctness conditions** that fluent prose can fake — an argument can be
invalid, rest on a false premise, or equivocate on a key term, and competent
writing sails right past all three.

So this system is **not a truth oracle**. It does not declare winners. Its
deliverable is a **disagreement map**: the steelmanned version of each serious
position, and the single load-bearing premise where they part ways. A human reads
that map and decides. The synthesizer's job is to locate the crux, not to break
the tie.

### The convergence hazard (read this before building anything)

Every agent here is the same underlying model. Left alone, they converge on
whatever reads as "competent philosophy" — a consensus-shaped output that is not
the same thing as truth. A persona label is a hat, not a different brain. The
**divergence protocol** below exists to fight this. If you skip it, you have built
an echo with name tags.

## The unit of work

One arguable thesis: a claim of the form **"X because Y"** that the author has a
stake in but is not certain of. Not a topic, not a corpus. If no serious
philosopher would take the other side, it is not a good seed — the council can't
fight over it.

## Pipeline

Each run targets one thesis and writes to `runs/<thesis-slug>/`.

| # | Stage | Agent | In → Out |
|---|-------|-------|----------|
| 0 | Frame | `framer` | topic/question → one thesis + classification |
| 1 | Reconstruct | `reconstructor` | thesis → explicit premises + conclusion |
| 1b | Cast | `caster` | argument → position assignments that span the verdict space |
| 2 | Council | 4 fixed personas (+ summoned specialists) | argument + cast → one committed position each |
| 3 | Functional pass | functional roles | argument + positions → form/grounding reports |
| 4 | Synthesize | `synthesizer` | all of the above → the crux map |
| 5 | Communicate | `communicator` | crux map → lay-readable version |

### The divergence protocol (stage 2 — orchestration, not an agent)

The main session runs the council like this. **Do not shortcut it.**

1. **Cast first (stage 1b).** Do not improvise the assignments. Run the `caster`,
   which reads the argument and hands each seat a position to defend tied to a
   specific premise — guaranteeing the council spans the verdict space (≥2 seats
   oppose the thesis, ≥1 defends it, no two seats making the same move). This is
   what makes divergence a property of the system, not of the orchestrator's
   per-run cleverness. Paste each seat's one-line brief from `01b-casting.md`
   verbatim into that seat's prompt.
2. **Assign, don't elicit.** Following the cast, hand each persona its position to
   defend. Do not ask "what do you think of thesis X." Say: "You hold that X is
   false because [the assigned brief]. Build the strongest case." Adversarial
   assignment beats persona flavor.
3. **Draft blind.** Spawn the personas so that **no persona sees another's
   output**. Launch them in one batch; give each only the reconstructed argument
   and its own brief, never a sibling's draft. Context contamination is what
   collapses them.
4. **Attack round (full mode only).** Only after all blind drafts exist, run one
   round where each persona reads the others and appends a `## Attack round`
   section **to its own file** (not a shared file — parallel writes to one file
   collide).
5. **Commit.** Each persona ends with a one-sentence verdict. The hedge-free
   commitment stops the model's instinct to write balanced both-sides prose that
   secretly agrees with everyone.

### Fixed council vs summoned specialists

- **Fixed council (always runs):** `naturalist`, `rationalist`, `phenomenologist`,
  `contemplative`. Chosen for maximal mutual disagreement, not topical coverage.
- **Summoned per question:** instantiate `domain-specialist` one or more times with
  a named position relevant to the thesis (e.g. a Chalmers-style dualist for
  philosophy of mind). The orchestrator passes the position in the prompt.
- **Note on the author's own view:** the author is a pragmatist. Do **not** seat a
  pragmatist on the council — that chair is an echo of the user's frame. The
  council should be the people the author would argue with, not nod along with.

### Functional roles (in parallel)

These carry no worldview. They check form and grounding, which is mechanizable;
they do **not** check truth, which is not. Be honest about that limit. Which of
them run depends on the mode (see Modes below).

- `hidden-premise-excavator` — surfaces the unstated premises the argument needs.
- `counterexample-generator` — manufactures cases the thesis must survive.
- `distinction-watchdog` — catches equivocation; polices bleed between
  phenomenological, metaphysical, and empirical claims.
- `self-application-tester` — does the thesis survive being applied to itself?
  ("All truths are relative" fails this. Verificationism fails this.) This is the
  closest thing here to a unit test that actually runs.
- `attribution-checker` — the retrieval role. Verifies that positions attributed
  to named philosophers match what they actually wrote, against fetched sources.

## Modes

Pick a mode per run. A full run of the pipeline costs well over a million tokens
across ~19 agents; light mode is for quick passes and triage.

| | **Light** | **Full** |
|---|---|---|
| Cast | yes | yes |
| Council blind drafts | yes (4 fixed + specialists) | yes (4 fixed + specialists) |
| Council attack round | **skipped** | yes |
| Functional roles | `self-application-tester`, `distinction-watchdog`, `attribution-checker` | all five |
| Synthesize + Communicate | yes | yes |

**Light** keeps the three roles that catch the cheapest fatal errors — a
self-refuting thesis, an equivocation, a fabricated attribution — and drops the
attack round plus the two generative roles (`hidden-premise-excavator`,
`counterexample-generator`), which add depth but rarely change the verdict on a
quick pass. **Full** is the default for a thesis the author actually has a stake
in. When unsure, run light first; promote to full if any of these fire:
- the crux map is thin, or
- the council looks like it's converging, or
- **the reconstructor flags a suppressed premise that looks load-bearing.** Light
  mode drops the `hidden-premise-excavator`, so it has no machinery to develop a
  buried premise into the crux — and a buried premise is exactly where the real
  crux can hide (in the moral-responsibility run, the excavator relocated the whole
  crux to a hidden self-authorship premise). When the stated argument leans on an
  unstated assumption, light mode is flying blind; promote it.

## Retrieval

Retrieval is first-class in v1, on both sides:
- **Generation side:** council personas and specialists have `WebSearch`/`WebFetch`
  and are instructed to ground claims in primary text, not paraphrase from memory.
- **Verification side:** `attribution-checker` independently checks attributions
  against sources. Ungrounded personas misquote dead philosophers confidently;
  this is the cheapest large win in the system.

## Verdict tokens

Critic-style agents emit a verdict token on the **first line**, mirroring the game
studio's gate format, so the orchestrator can read the verdict without parsing
prose. Examples: `SELF-APPLICATION: SURVIVES`, `DISTINCTION: CONCERNS`,
`ATTRIBUTION: VERIFIED`.

## Run output layout

```
runs/<thesis-slug>/
  00-thesis.md          framer
  01-reconstruction.md  reconstructor
  01b-casting.md        caster — seat assignments + span check
  02-council/
    naturalist.md       blind draft + verdict, then "## Attack round" appended (full mode)
    rationalist.md
    phenomenologist.md
    contemplative.md
    specialist-*.md      summoned, if any
  03-functional/
    hidden-premises.md
    counterexamples.md
    distinctions.md
    self-application.md
    attribution.md
  04-crux.md            synthesizer
  05-lay.md             communicator
```

## Theses

Candidate seeds live in `theses/`. The current tuning harness is
`theses/cosmic-backstop.md`.
