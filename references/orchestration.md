# Orchestration: running the build with subagents

## Default behavior: delegate. This is not optional.

**For any design-system build past roughly 6 components or 8 files, you MUST run as an
orchestrator and fan work out to subagents.** This is the default execution mode, not a
technique to consider if you feel like it.

Building serially in one session is a failure mode, not a safe conservative choice:

- It is slower by a large multiple on work that is genuinely independent.
- It burns the orchestrator's context on implementation detail that should never enter it.
  You need that context for decisions and reconciliation.
- **Quality degrades on the later components**, because by the time you reach component 40
  the context is full of the previous 39 and attention to the spec has decayed.

A full inventory build is roughly 70 components. That is not a serial job.

**Announce the plan before dispatching.** Tell the user which agents you are creating, what
each one owns, and what comes back. Then dispatch.

### Before every fan-out, run this checklist

1. Is the spec written down and stable? If it is still changing, do not dispatch.
2. Does every agent have **exclusive file ownership**, with zero overlap?
3. Does each prompt carry the spec inline, the token and component allowlist, the
   constraints that will judge it, and the exact return format?
4. Is the batch 3-5 agents?
5. Do I know how I will verify each return?

If any answer is no, fix it before dispatching. A bad dispatch costs more than the
serial work it replaced.

### A subagent cannot see your conversation

It has no access to the user's messages, your reasoning, or the decisions you made three
steps ago. **Anything it needs must be in its prompt.** "Read the skill and figure it out"
is not context, it is an invitation to guess. Inline the spec excerpt.

### A worked fan-out for a real build

There are two fan-outs in a full build: one for the HTML preview, one for the React
components.

**HTML preview batch** (after tokens and naming are locked, before approval):

| Agent | Owns (section, returns a fragment) |
|---|---|
| 1 | Foundations: color, type, spacing, radius, elevation, motion, icons |
| 2 | Atoms A-L |
| 3 | Atoms M-Z |
| 4 | Molecules A-M |
| 5 | Molecules N-Z |
| 6 | Organisms |
| 7 | Layout primitives and composed pages |
| **Orchestrator** | The shell, the stitch, verification |

**React component batch** (after approval). Copy the shape.

| Agent | Owns exclusively | Returns |
|---|---|---|
| 1 | `ui/{button,icon-button,link,text,heading,icon}.tsx` + their stories | file paths, gaps hit |
| 2 | `ui/{badge,status-dot,avatar,divider,spinner,skeleton}.tsx` + stories | same |
| 3 | `ui/{checkbox,radio,switch,toggle-button,slider,progress-bar}.tsx` + stories | same |
| 4 | `ui/{text-input,text-area,number-input,select,field}.tsx` + stories | same |
| **Orchestrator** | `tokens.json`, `registry.json`, the preview, all naming, reconciliation | - |

Note what the orchestrator kept: every shared, single-writer artifact. Note what the agents
got: disjoint file sets that can fail and be retried independently.

Then the next batch does molecules, the next does organisms. Layer order matters, because
molecules import atoms and organisms import both.

A full canonical inventory (24 atoms, 27 molecules, 16 organisms, 6 layout primitives) is
roughly 15 agent-batches of 4-5. Plan for that shape rather than discovering it at
component 12.


**The orchestrator owns the user relationship, every shared artifact, and every decision.
Subagents own bounded, non-overlapping implementation work and report back.**

---

## What the orchestrator never delegates

- **Discovery and the user conversation.** Questions, defaults, corrections, approval.
- **The token source.** One file, one writer. Parallel edits to it are a guaranteed
  conflict.
- **The registry / allowlist.** Same reason.
- **The design source file**, if one exists.
- **The visual philosophy and any naming decision.** Five subagents naming things
  independently produces five vocabularies.
- **The approval gate.** Never let a subagent decide the preview is good enough.
- **Reconciliation.** When returns conflict, the orchestrator resolves.

Rule of thumb: **if two agents could write the same file, only the orchestrator writes it.**

---

## What parallelizes well

| Work | Parallel? | Notes |
|---|---|---|
| Research and precedent gathering | Yes | Independent sources, no shared output |
| Component authoring | Yes | One agent per component or small cluster, exclusive file ownership |
| Story files | Yes | Same ownership rule, one per component |
| Per-component a11y audit | Yes | Read-only, reports back |
| Contrast audit | No | Single pass over the whole token set, cheaper in one place |
| Token authoring | No | Single shared file |
| Registry updates | No | Single shared file |
| HTML preview **sections** | Yes | Agents return **fragments**, never write the file. Orchestrator writes the shell first, then stitches. See below. |
| HTML preview **shell and stitch** | No | Token CSS, theme toggle, base classes, concatenation. Orchestrator only. |
| Naming decisions | No | Consistency is the entire point |

---

## Fanning out a single large file

The HTML preview is one file containing roughly 73 components plus foundations plus
composed pages. Too large for one agent, and quality visibly degrades toward the end of a
long serial build. But it is one file, so the usual exclusive-file-ownership rule cannot
apply directly.

**The pattern: ownership by section, delivery by fragment.**

1. **Orchestrator writes the shell alone, first.** Every token as a CSS custom property in
   both themes, the theme-toggle script, shared base and utility classes, section
   scaffolding, and the checklist. This shell is the contract.
2. **Agents receive the shell and return HTML fragments**, not files. Markup plus any
   component-specific script for their section only.
3. **The orchestrator concatenates.** No agent ever writes the target file. This is what
   makes a single-file fan-out safe, and it is the only reason it works.
4. **Orchestrator verifies the stitch**: every checklist item present, theme toggle working
   across all sections, no invented tokens or colors, interactions actually functional.

**Close inconsistency structurally, do not rely on a thick shell.** The shell must define
every component base class, not just tokens, and the rule for every agent is: **return
markup only, never CSS, never inline styles.** An agent that cannot author a style cannot
invent one, and the only remaining move when something is missing is to report the gap.

Then verify mechanically rather than by eye: extract every class used in the stitched file
and diff it against the classes the shell defines. Anything not in the shell is invention,
and the diff names it. Plus greps for `<style`, inline `style=`, and color literals, all of
which must be zero outside the shell.

Sequence it as **shell, then one reference section you build yourself, then fan out.**
Models weight a nearby concrete example far above stated instructions, and building one
section first also proves the shell is sufficient before seven agents discover it is not.
See `delivery.md`.

The same pattern generalizes to any large single artifact: a docs page, a long spec, a
generated index. Own the skeleton centrally, fan out the contents, stitch centrally.

## Sequence: phases are serial, work inside a phase is parallel

```
Discovery            orchestrator only, with the user
Inventory            orchestrator, or 2-3 agents on different surfaces
Visual language      orchestrator decides, subagents may research
Tokens               orchestrator authors, one writer
Naming               orchestrator locks it before any component exists
Components           FAN OUT, 3-5 agents, exclusive file ownership
Stories              FAN OUT, same ownership map
Enforcement          orchestrator wires it, before generation begins
Preview              orchestrator assembles
Approval             orchestrator with the user
Storybook            FAN OUT for story files, orchestrator for config
Skill file           orchestrator generates
Evaluation           a fresh, context-free agent, see evaluation.md
```

**Do not cross a phase boundary in parallel.** Components dispatched before naming is
locked will each invent their own vocabulary, and you will pay for it in a rename that
touches everything.

---

## Spec first, then fan out

The single highest-leverage practice: **decide and write everything down before
dispatching, then give each subagent the relevant excerpt inline.**

Subagents should implement from a spec, never re-derive from the source. If five agents
each measure or interpret independently, you get five different answers and no way to tell
which is right. One extraction, one written spec, then parallel implementation off it.

Each dispatch should carry:

1. **The exact files it owns**, and an instruction to touch nothing else.
2. **The spec excerpt** for its piece, inline. Do not make it go looking.
3. **The token names and component allowlist** it may use. It cannot discover these.
4. **The constraints that will judge it**: the lint rules, the no-invention rule, the a11y
   bar.
5. **The exact return format** you expect.

A subagent that has to search for context will guess, and guessing is what produces drift.

---

## Failure modes, all observed in practice

These are not hypothetical. Every one of these happened while building this skill.

**1. Subagents drift toward doing the whole job.** Given a scoped research task, multiple
agents independently decided to write the final deliverable instead, each producing a
competing version. Explicit instructions not to were ignored more than once.

*Mitigation:* state the deliverable format in the imperative and repeat it at the end of
the prompt. Name the files they may write, or say plainly that they must write none. Then
verify on return rather than trusting the report.

**2. Concurrent writes to the same path.** Three agents wrote to one filename. The result
was one surviving file, silently, with no error and no indication whose version won.

*Mitigation:* assign exclusive file ownership in writing, and never give two agents the
same output path. If a shared file must change, the orchestrator changes it.

**3. Reports that do not match the artifacts.** Agents reported success while having
produced something adjacent to what was asked, or reported word counts and file states
that did not survive inspection.

*Mitigation:* the same rule as `evaluation.md`. **Score the artifact, not the self-report.**
After every batch, list the files, check sizes, grep for the thing that should be there,
and diff if it matters.

**4. Batch failures.** Rate limits and transient errors killed a portion of a parallel
batch mid-run.

*Mitigation:* keep batches to 3-5. Expect partial failure. Make each task independently
retryable, which exclusive file ownership already gives you. Never make agent B depend on
agent A's in-flight output.

**5. Duplicated content after a merge.** Recovering from collisions produced files with the
same section written twice in slightly different words.

*Mitigation:* after any reconciliation, check for repeated headings and near-duplicate
paragraphs before moving on.

**6. Stale shared state.** When the orchestrator updates a spec but not the design source,
agents correctly follow the spec and the parity check then reports drift only the
orchestrator can clear.

*Mitigation:* update every shared artifact in the same pass, before dispatching.

---

## Verification after every batch

Non-negotiable, and cheap:

- List the files that changed. Do they match the ownership map exactly?
- Did anything outside the brief change? A `git status` answers this in one line.
- Do the gates still pass: typecheck, lint, token check, registry check?
- Spot-read one returned file in full rather than trusting the summary.

Fix drift immediately. A wrong component that ships into the registry becomes precedent,
and the next batch will copy it.

---

## Writing a good dispatch

A component-authoring dispatch should be roughly this shape:

```
Build exactly these files, and nothing else:
  components/ui/<name>.tsx
  components/ui/<name>.stories.tsx

Spec for this component (implement exactly, do not re-measure):
  <inline spec excerpt: anatomy, variants, states, tokens per part>

You may use only these tokens: <list>
You may import only from this allowlist: <list>

Constraints that will be checked:
  - no raw hex, rgb, or off-scale spacing
  - semantic tokens only, never primitives
  - every interaction state present
  - a11y addon must pass
  - if you need something that does not exist, STOP and report it as a gap.
    Do not invent it.

Return: the file paths you wrote, and any gap you hit. Write no other files.
```

The last two lines matter most. **An agent that hits a missing component and improvises one
has produced working code that is not your design system**, and it will pass typecheck and
render correctly while doing it. Make reporting the gap the explicitly correct move.

---

## When not to use subagents

- Fewer than about eight components. The coordination overhead exceeds the gain.
- Anything requiring a consistent judgment call across items. Do it in one place.
- The first component of a new pattern. Build one yourself, establish the shape, then fan
  out using it as the reference.
- Anything touching the user relationship.

Parallelism buys throughput on independent work. It buys nothing on decisions, and it costs
you coherence.
