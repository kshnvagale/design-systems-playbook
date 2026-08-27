# Orchestration: running the build with subagents

A design system build is wide but shallow in places (thirty components, each independent)
and narrow but deep in others (one token source everything depends on). That shape suits
an orchestrator plus subagents, but only if the split respects which is which.

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
| HTML preview assembly | No | One file, needs a coherent whole |
| Naming decisions | No | Consistency is the entire point |

---

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
