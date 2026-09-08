# Handoff: Storybook, progress tracking, and the generated skill file

What happens after the previews are approved and the foundation is chosen. For the
preview and approval sequence, see `preview.md`.

## 1. Storybook implementation

Every component ships with a story file. A component without one is not done.

### Per-component definition of done

- **CSF3 format**, typed: `Meta<typeof Component>` and `StoryObj<typeof meta>`.
- **`tags: ['autodocs']`** so a docs page is generated rather than hand-maintained.
- **`argTypes` for every prop**, with an explicit control type (`select`, `boolean`,
  `radio`, `text`, `number`, `color`) and a `description`. Group related props with
  `table.category`. A prop with no control is a prop nobody can explore.
- **One story per variant.** If `Button` has primary, secondary, and tertiary, that is
  three named stories, not one story with a control the reviewer has to discover.
- **One story per state**: default, hover, focus, pressed, disabled, loading, error,
  selected, empty. Where a state cannot be forced by props, use a `play` function.
- **A `play` function for anything interactive**, using `canvas` and `userEvent` from
  `storybook/test`. Always `await` userEvent calls or the Interactions panel cannot log
  them.
- **Actions wired** with `fn()` so events appear in the Actions panel.
- **Real content in args.** Same rule as the preview.
- **Accessibility addon passing**, with any deliberate exception documented in the story.
- **Works in both themes**, via `@storybook/addon-themes` (`withThemeByClassName` or
  `withThemeByDataAttribute`, matching how your tokens are applied) plus a `globalTypes`
  toolbar entry.

### Global configuration

- Theme switcher in the toolbar, applying real theme tokens.
- Viewport addon configured with your actual breakpoints, not the defaults.
- A docs landing page covering foundations: color, type, spacing, motion, and the visual
  philosophy.
- Motion enabled. Do not globally disable transitions for snapshot stability; handle that
  in the visual-regression config.

### Publish it

A local Storybook helps nobody outside the team. Publish it and treat that URL as the
system's address. Storybook emits an `index.json` listing every story and generates prop
metadata from your types, which is what makes it machine-readable downstream.

---

## 2. PROGRESS.md: the build's own state file

A full build is roughly 73 components across several phases, fanned out to subagents,
spanning multiple sessions. Nothing about that survives a context window. **Create a
`PROGRESS.md` at the start and keep it current.**

It does three jobs:

1. **Resumability.** A new session, or the same session after compaction, reads one file
   and knows exactly what exists and what does not.
2. **Coordination.** Parallel agents work against one shared picture of scope.
3. **Verification surface.** A ticked box that does not match the codebase is a drift
   signal, and a cheap one to check.

### Create it right after the inventory is settled

Every item from the canonical inventory becomes a checkbox, grouped by phase and layer:

```markdown
# ShiftCart Design System - Build Progress

Updated: 2026-08-27 | Phase: components | Gate 1: passed | Gate 2: passed

## Phase 1: Foundations
- [x] Discovery summary approved
- [x] Visual philosophy written (5 statements)
- [x] Color: 72 primitives, 62 semantic
- [x] Type: 11 roles
- [x] Spacing, radius, elevation, motion
- [x] Contrast audit: 0 failures across both themes

## Phase 2: Preview 1 (base system)
- [x] Shell: tokens, theme toggle, base classes
- [x] Foundations section
- [x] Atoms (24/24)
- [x] Molecules (27/27)
- [x] Organisms (16/16)
- [x] Layout primitives (6/6)
- [x] GATE 1 APPROVED 2026-08-26

## Phase 4: React components
### Atoms (18/24)
- [x] Button          variants: 4  states: 6  story: yes  a11y: pass
- [x] Icon Button     variants: 3  states: 6  story: yes  a11y: pass
- [ ] Slider          BLOCKED: needs range token decision
- [ ] Kbd
...

## Open gaps
- Slider: no range-fill token exists. Decide before building.
- Toast: stacking behavior undefined beyond 3 concurrent.
```

Carry per-component detail (variants, states, story, a11y) rather than a bare tick. A
ticked box that only means "a file exists" is worth very little.

### Who writes it

**The orchestrator writes it. Subagents do not.**

This follows the single-writer rule in `orchestration.md`, and it matters more here than
anywhere else: `PROGRESS.md` is one file that every agent in a batch would touch at the
same time. Concurrent read-modify-write on a markdown file produces lost updates silently,
with no error and no indication whose version won. That is the same failure mode as two
agents writing the same component file.

The protocol:

1. Each subagent **returns a structured completion list**: which items it finished, with
   the per-item detail, plus any gap it hit.
2. The orchestrator **verifies** against the actual files.
3. The orchestrator **ticks the boxes**.

The verification step is not ceremony. Agents have been observed reporting success on work
that did not match the artifacts, so ticking only what you confirmed is what keeps the file
trustworthy. An unreliable progress file is worse than none, because it gets believed.

**If you want agents writing directly**, the safe variant is one status file per agent
(`.progress/atoms-a-l.md`) that the orchestrator merges into `PROGRESS.md`. No shared
write, agents still self-report. Slightly more moving parts, same guarantee.

### Keep it current at phase boundaries

Update after every batch returns and at every gate. A progress file updated once at the
start is a plan, not a progress file.

---

## 3. The generated skill file

When the system is built, generate **one skill file** so a designer or an agent can use it
without rediscovering it. One file, not a folder.

### The generation rule that matters

**Split it into generated and hand-written parts, and mark the boundary in the file.**

- **Generated from the system**: token inventory, component allowlist with import paths and
  variants, the scales. Emitted by a script from the token source and the registry, so it
  physically cannot drift from what was built.
- **Hand-written**: visual philosophy, judgment calls a linter cannot make, known traps.
  The taste layer, which no generator produces.

Regenerating must be a single command, run whenever a component or token changes.

### Required contents

1. **Where the system lives.** The published Storybook URL, stated as source of truth.
2. **The hard rules** (below).
3. **The component allowlist**: every component, import path, variants. Generated.
4. **The token inventory**: semantic tokens by group, plus spacing, radius, type scales.
   Generated.
5. **The visual philosophy**: the three to five falsifiable sentences from discovery.
6. **Judgment calls**: Badge vs Chip, when a surface gets a border, density by surface
   type, where delight is allowed and where it is banned.
7. **Known traps** specific to this codebase.
8. **How to add something**, so a genuine gap becomes a contribution rather than an
   improvisation.

### The hard rules, stated verbatim

Put these at the top, where a model reliably reads them:

```
- The design system is published at <STORYBOOK_URL>. That is the source of truth.
- Fetch component APIs, props, variants, and tokens FROM Storybook. Do not infer them
  from screenshots, memory, or another project.
- Never invent a component. If it is not in the allowlist below, it does not exist.
- Never invent a color, spacing value, radius, type size, or motion value. If it is not
  a token, it does not exist.
- Never override a component's visual props on an instance. If you need a different
  appearance, the component is missing a variant.
- If something you need is genuinely missing, STOP and report it as a system gap.
  Do not improvise a replacement.
```

State them as prohibitions with a named consequence. "Prefer using design tokens" is a
suggestion a model weighs against other considerations. "If it is not a token, it does not
exist" is not.

### How design tools and agents fetch from it

Same always-on plus on-demand split used everywhere else in this skill:

- **Always-on**: the generated skill file, carrying the allowlist and token inventory
  inline. Small, cheap, present before the model writes anything. Foundations cannot be
  fetched on demand because the model needs them before it knows it needs them.
- **On-demand**: Storybook for detail. Its `index.json` enumerates every story and autodocs
  metadata carries prop types. Point a Storybook MCP server or fetch tool at that.

### Size budget, and what "thin" actually means

`ai-agents.md` warns that a rules file which outgrows what a model reliably attends to is
worse than none. That is correct, and it is not in conflict with shipping the allowlist
inline, but the line between them has to be stated or the two rules read as contradictory.

**The split is names inline, detail fetched.**

| Inline in the always-on file | Fetched on demand |
|---|---|
| Component names and import paths | Full prop APIs and types |
| Variant names per component | Per-variant behavior and edge cases |
| Semantic token names by group | Token values and the primitive chain |
| The scales (spacing, radius, type roles) | Usage examples and compositions |
| Visual philosophy, judgment calls, traps | Anything a model can look up once it knows the name exists |

The reasoning: **a model cannot fetch what it does not know exists.** If the allowlist is
behind a tool call, the agent has to already suspect a component is there before it asks,
and when it does not suspect, it invents. Names are cheap and they are the thing that
prevents invention. APIs are expensive and only needed once a name is chosen.

**Budget: keep the always-on file under roughly 5,000 tokens.** For a 73-component system
that is achievable: names and paths run about 1,500 tokens, semantic token names about 800,
scales about 300, philosophy and judgment about 1,000. If you are over budget, cut prose
and examples first, never the allowlist.

If your system is large enough that the names alone blow the budget, that is a signal to
split the registry by layer and load the relevant one, not to move names behind a fetch.

Push everything else behind the fetch.

### Also ship it as `AGENTS.md`

Symlink or copy the generated file to `AGENTS.md` at the repo root so coding agents pick it
up automatically without configuration. Same content, conventional filename.
