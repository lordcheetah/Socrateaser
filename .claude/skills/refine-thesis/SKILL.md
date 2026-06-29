---
name: refine-thesis
description: Continue or refine an EXISTING Socrateaser run instead of starting over. The author reacts to a finished run — clarifies what they meant, says what they accept or reject, asks to push a premise harder, or adds a position — and this skill reuses every artifact the reaction does not touch, re-running only what changed, then re-synthesizes. Use when the user says "refine that run", "continue X", "I disagree with the naturalist", "by Y I actually meant Z", "push harder on the crux", or reacts to a run's crux map.
---

# Refine Thesis

Continue a run without restarting it. You (the main session) are the orchestrator.
The point is **reuse**: re-run only the agents the author's reaction actually
touches, feed them their prior work, and tell them to extend it — not redo it.

## The load-bearing rule — test, do not accommodate

This is the rule that keeps Socrateaser a disagreement-mapper and not a flattery
machine. **The author's reaction is argumentative input to be tested, never a
verdict the council must adopt.** Never edit a council position so it agrees with
the author. Concretely:
- The author **agreeing** with a position prunes effort there — it does not make the
  position "win."
- The author **disagreeing** becomes a *steelmanned challenge* the named seat must
  answer or concede — the seat keeps its commitment and defends it.
- Only a genuine **reframe** ("I actually meant Y") changes the thesis itself.
- The author is still **never seated** (no pragmatist chair). If the author's own
  view needs a voice, summon it as a specialist the council attacks.

## Inputs

- **The run** (required): the slug of an existing run under `runs/<slug>/`.
- **The reaction** (required): the author's free-text response to the run.

## Step 1 — Load context

Read, in this order: `runs/<slug>/04-crux.md` (the summary — orient here first),
then `00-thesis.md`, `01-reconstruction.md`, `01b-casting.md`, the council drafts in
`02-council/`, and the functional reports in `03-functional/`. Note the run's
**mode** and **run title** from the headers.

## Step 2 — Route the reaction

Classify the reaction into one or more of the five types below. State your routing
to the user in one or two lines before executing — it is the author's chance to
correct you. A reaction can be several types at once (e.g. a reframe plus a
challenge); handle each.

| Type | Trigger | What re-runs | What is reused |
|---|---|---|---|
| **Reframe** | "by X I meant Y"; the thesis was mis-stated | framer (revise `00`, retitle if needed) → reconstructor (`01`) → caster (`01b`) → only seats whose brief changed | unaffected seats, functional reports still valid |
| **Disagree** | "I don't buy the naturalist's move" | the named seat only — a one-seat challenge round | every other seat and report |
| **Agree / concede** | "the realist convinced me on P3" | nothing in the council | all drafts; the concession is passed to the synthesizer as settled |
| **Deepen** | "push harder on the meta-crux / SP-I" | the seats tied to that premise + the relevant functional role | everything else |
| **Add position** | "what about a higher-order view?" | one new `domain-specialist` (blind), then integrate | all existing drafts |

If a **reframe** is deep enough that most seats' briefs change, say so and offer to
run it as a fresh run instead — refinement is for surgical change, not a rewrite.

## Step 3 — Execute the minimal re-run

For every re-invoked agent: give it **its own prior file plus the relevant existing
artifacts**, and instruct it explicitly: *"Do not start over. Reuse your prior
reasoning and citations; research only what the reaction newly requires. Append a
new section; do not rewrite what stands."*

- **Council seats** append a section to their **own file** (never a shared file):
  ```markdown
  ## Refinement <N> — <short label>
  > **Author input:** <quote the reaction>
  > **Routing:** <type>
  <the seat's revised case, defense, or concession — and an updated one-line VERDICT
  if it moved>
  ```
- **New specialists** (add-position) get a new `02-council/specialist-<slug>.md`
  with the standard council header.
- **Reframe** cascades through framer/reconstructor/caster as in a normal run, but
  reuse and revise the existing files rather than blank ones.
- Honor the run's **mode** (a refinement can be lighter than the run — a single
  challenge round needs no full functional sweep). Functional roles re-run only if
  the reaction touches what they check (e.g. a reframe → re-check distinctions;
  a new attribution → re-check attribution).

## Step 4 — Re-synthesize and communicate (always)

Re-run the **synthesizer** → `04-crux.md` and the **communicator** → `05-lay.md`.
These are cheap relative to the council and they are the deliverable. Tell the
synthesizer what changed and what the author conceded, and have it note the delta
from the prior crux (the prior `04-crux.md` is preserved in git history). It remains
**forbidden from declaring a winner**, and a concession narrows the map rather than
settling it.

## Step 5 — Log it

Append to `runs/<slug>/REFINEMENTS.md` (create it the first time, with the standard
pipeline header):
```markdown
## Refinement <N> — <label>
- **Author input:** <quote>
- **Routing:** <type(s)>
- **Re-ran:** <agents>
- **Reused untouched:** <agents>
- **Crux delta:** <one line: how the crux map changed, or "unchanged">
```

## After the refinement

Report: the routing you used, what re-ran vs. what was reused, the **crux delta**,
and any new CONCERNS tokens. Link `04-crux.md` and `05-lay.md`. If the reaction was
a deep reframe you handled surgically, note what might still be stale.

## Reminders

- **Test, don't accommodate** — the rule above is the whole point; re-read it if a
  reaction tempts you to make a seat capitulate to the author.
- **Reuse, don't restart** — every re-invoked agent gets its prior file and is told
  to extend, not redo.
- **The deliverable is still a disagreement map.** A refinement sharpens the crux or
  moves it; it does not crown a winner.
