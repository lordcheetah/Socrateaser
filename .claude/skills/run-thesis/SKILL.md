---
name: run-thesis
description: Run the full Socrateaser pipeline on one arguable thesis or topic — frame, reconstruct, cast, run the council under the divergence protocol, run the functional rigor roles, synthesize the crux map, and communicate it for a lay reader. Use when the user wants to work out a philosophical claim ("run this through Socrateaser", "what's the crux of X", "stress-test this thesis"). Takes a topic/question/thesis plus an optional mode (--light / --full).
---

# Run Thesis

Orchestrate one philosophy run end to end. You (the main session) are the
orchestrator. This skill encodes the procedure; **the divergence protocol in it is
load-bearing — do not shortcut it, or the council collapses into one voice wearing
name tags.** Read `CLAUDE.md` first if you have not this session.

## Inputs

- **The seed** (required): a topic, question, or rough thesis from the user. If
  they point at a file in `theses/`, read it.
- **Mode** (optional): `--light` or `--full`. If unspecified, choose per *Mode
  selection* below and tell the user which you picked and why.

## Mode selection

Default to **light** for a quick pass or triage; **full** for a thesis the user has
a real stake in. If unsure, run light, then **promote to full** if any fire:
1. the crux map comes out thin, or
2. the council looks like it is converging, or
3. the reconstructor flags a suppressed premise that looks load-bearing (light mode
   drops the `hidden-premise-excavator`, so it is blind exactly where a buried
   crux hides — promote).

| Stage | Light | Full |
|---|---|---|
| Cast | yes | yes |
| Council blind drafts | yes | yes |
| Council attack round | **skip** | yes |
| Functional roles | `self-application-tester`, `distinction-watchdog`, `attribution-checker` | all five |
| Synthesize + Communicate | yes | yes |

## Setup

Derive a short kebab-case **slug** from the thesis (e.g. `cosmic-backstop`). All
outputs go under `runs/<slug>/`. Use absolute paths in every agent prompt.

Invoke each stage with the matching registered agent type via the Agent tool
(`subagent_type: framer`, `reconstructor`, `caster`, `naturalist`, `rationalist`,
`phenomenologist`, `contemplative`, `domain-specialist`, `hidden-premise-excavator`,
`counterexample-generator`, `distinction-watchdog`, `self-application-tester`,
`attribution-checker`, `synthesizer`, `communicator`). Tell each agent its inputs,
its output path, and (for council seats) its blind constraint.

## Procedure

### Stage 0–1b — Frame, Reconstruct, Cast (sequential; each needs the prior)

1. **framer** → `runs/<slug>/00-thesis.md`. One arguable "X because Y" +
   classification + contestability + pseudo-dispute check + suggested specialists.
   If the framer reports the thesis is uncontestable or dissolves under a
   distinction, **stop** and report that — a bad seed is a real result.
2. **reconstructor** → `runs/<slug>/01-reconstruction.md`. Premises/conclusion,
   validity, where the weight sits, charitable variants. Note any suppressed
   premise it flags (feeds the mode-promotion check).
3. **caster** → `runs/<slug>/01b-casting.md`. Seat assignments + span check +
   specialist recommendations.

Then **read `01b-casting.md` yourself.** Confirm the span check passed (≥2 seats
oppose the thesis, ≥1 defends it TRUE, no duplicate moves). You will paste each
seat's one-line brief **verbatim** into that seat's prompt next.

### Stage 2 (blind) — Council + argument-only functional roles (ONE parallel batch)

Launch all of the following in a **single message** (concurrent). This is the blind
draft: each council seat gets ONLY `00-thesis.md` and `01-reconstruction.md` and its
own pasted brief — **never a sibling's draft.** Tell every council seat explicitly:
"BLIND RULE: do not read any file in `02-council/`."

- The four fixed seats — `naturalist`, `rationalist`, `phenomenologist`,
  `contemplative` — each with its verbatim brief from the cast, writing to
  `runs/<slug>/02-council/<seat>.md`. Each ends with a one-sentence `VERDICT:` line.
- One `domain-specialist` per specialist the caster recommended, each given its
  named position in the prompt, writing to
  `runs/<slug>/02-council/specialist-<slug>.md`.
- The argument-only functional roles (they need only thesis + reconstruction):
  - **full mode:** `hidden-premise-excavator`, `counterexample-generator`,
    `self-application-tester`
  - **light mode:** `self-application-tester` only
  writing to `runs/<slug>/03-functional/<role>.md`, each with its verdict token on
  the first line.

Tell council seats to ground claims in cited sources (they have WebSearch/WebFetch)
and remind the `contemplative` of the translation-fidelity rule in its definition.

### Stage 2 (attack) + council-dependent functional roles

These need the council drafts to exist, so run them after Stage 2 blind completes.
Launch as one parallel batch:

- **Attack round — FULL MODE ONLY.** Re-invoke each council seat (and specialist),
  now told to read the *other* drafts in `02-council/` and **append** a
  `## Attack round` section to **its own file** (never a shared file — parallel
  writes collide). Keep attacks tight (≤120 words per rival).
- `distinction-watchdog` → `runs/<slug>/03-functional/distinctions.md` (both modes).
  Reads thesis, reconstruction, and all council drafts.
- `attribution-checker` → `runs/<slug>/03-functional/attribution.md` (both modes).
  Reads all council drafts; verifies attributions against fetched sources.

### Stage 4–5 — Synthesize, Communicate (sequential)

4. **synthesizer** → `runs/<slug>/04-crux.md`. Reads everything. Produces the crux
   map. **Forbidden from declaring a winner.** Tell it which functional reports
   exist (in light mode there is no `hidden-premises.md` / `counterexamples.md` —
   point it at the reconstructor's flagged suppressed premise instead). Tell it the
   author is a pragmatist with a stake; it must place that view and the single
   premise it would have to defend, without flattering it.
5. **communicator** → `runs/<slug>/05-lay.md`. Reads the crux map, invokes the
   `anti-ai-slop-writing` skill, translates for a non-specialist without inventing
   a resolution the synthesizer left open.

## After the run

Report to the user: the **crux** (one or two sentences from `04-crux.md`), the
**settleability** verdict, where the author's view sits, and any **CONCERNS** tokens
the functional roles raised (a failed self-application, an equivocation, an
unverifiable attribution). Link the run directory. If you ran light and any
promotion trigger fired, say so and offer to promote to full.

## Reminders

- **Assign, don't elicit.** Council seats defend a position; never ask "what do you
  think." The brief comes from the cast, pasted verbatim.
- **Blind first, attack second.** Contamination during the blind draft is what
  collapses divergence.
- **The deliverable is a disagreement map, not a verdict.** The synthesizer locates
  the crux; the human decides.
- A full run is ~19 agents and over a million tokens. Use light for triage.
