# Delivery: preview, approval, Storybook, and the generated skill file

The order here is deliberate and non-negotiable:

```
components authored -> single HTML preview -> USER APPROVAL -> Storybook -> generated skill file
```

**Do not build Storybook before the preview is approved.** Storybook is a real
implementation with config, addons, a build pipeline, and per-component story files. Doing
that work before anyone has agreed the components look and feel right means rebuilding it.
The HTML preview is cheap to produce and cheap to reject.

---

## 1. The single HTML preview file

### What it is

**One self-contained `.html` file** that a stakeholder opens by double-clicking. No server,
no `npm install`, no build step, no framework required on their machine. It is the approval
artifact.

### Hard requirements

- **Single file.** CSS inline or in a `<style>` block, JS in a `<script>` block. Fonts and
  the logo may be linked or embedded as data URIs. If it needs a second file to work, it is
  wrong.
- **Fully interactive, not a screenshot sheet.** Buttons depress, inputs focus and validate,
  toggles toggle, tabs switch, modals open and trap focus, dropdowns open and close on
  escape, steppers increment. If a component has behavior, the behavior works.
- **Motion running.** Real transitions on hover, press, enter, and exit, using the actual
  motion tokens. This is the only artifact where the user can feel the motion personality
  before committing to it, which is the entire reason motion is a discovery question.
- **Every variant and every state visible**, not just defaults. Rest, hover, pressed, focus,
  disabled, error, loading, empty. States that only appear on interaction should also be
  shown statically side by side so nothing has to be hunted for.
- **Working theme toggle.** Light and dark switch live, so both are reviewable in one pass.
  If there are brand or density axes, expose those too.
- **The token inventory rendered**, not just described: color swatches with names and hex,
  the type scale in situ, the spacing scale, radii, elevation, motion samples.
- **Real content.** Real product nouns, realistic string lengths, plausible numbers. Lorem
  ipsum hides every layout problem that matters.
- **The company logo in place**, in the header and anywhere the brand actually appears, so
  the system is reviewed as their product rather than as a generic kit.
- **A composed screen or two at the end.** Components in isolation always look fine. One
  realistic assembled screen is where slot collisions, density mistakes, and missing
  components become visible.

### Why a plain HTML file rather than a quick Storybook

It renders anywhere, including on a phone or in a shared Slack message. It survives being
emailed to someone who does not have Node. It costs an hour instead of a day. And it forces
the components to work as plain DOM and CSS before any framework abstraction hides a
problem.

---

## 2. The approval gate

Present the preview and ask for a decision, not for vague feedback. Ask specifically:

1. Does this feel like your product? If not, which part is wrong: color, type, density,
   radius, or motion?
2. Is the motion personality right: too flat, too much, or about right?
3. Any component that is missing, or too thin to carry your real content?
4. Anything here you would not use?

**Only after explicit approval, build Storybook.** If the answer is "close, but the density
is wrong," fix it in the HTML file and re-present. Iterating a single HTML file is cheap;
iterating a Storybook implementation is not.

Record the approval. It is the point at which the visual language stops being negotiable
and starts being a contract.

---

## 3. Storybook implementation

Every component ships with a story file. A component without one is not done.

### Per-component definition of done

- **CSF3 format**, typed: `Meta<typeof Component>` and `StoryObj<typeof meta>`.
- **`tags: ['autodocs']`** so a docs page is generated rather than hand-maintained.
- **`argTypes` for every prop**, with an explicit control type (`select`, `boolean`,
  `radio`, `text`, `number`, `color`) and a `description`. Group related props with
  `table.category`. A prop with no control is a prop nobody can explore.
- **One story per variant.** If `Button` has primary, secondary, and tertiary, that is three
  named stories, not one story with a control the reviewer has to discover.
- **One story per state**: default, hover, focus, pressed, disabled, loading, error,
  selected, empty. Where a state cannot be forced by props, use a `play` function.
- **A `play` function for anything interactive**, using `canvas` and `userEvent` from
  `storybook/test`, so the interaction is demonstrated and regression-tested in one artifact.
  Always `await` userEvent calls or the Interactions panel cannot log them.
- **Actions wired** with `fn()` so events are visible in the Actions panel.
- **Real content in args.** Same rule as the preview.
- **Accessibility addon passing**, with any deliberate exception documented in the story.
- **Works in both themes.** Use `@storybook/addon-themes` (`withThemeByClassName` or
  `withThemeByDataAttribute`, matching however your tokens are applied) plus a `globalTypes`
  toolbar entry, so every story is reviewable in every theme without editing code.

### Global configuration

- Theme switcher in the toolbar, applying real theme tokens.
- Viewport addon configured with your actual breakpoints, not the defaults.
- A docs landing page covering the foundations: color, type, spacing, motion, and the visual
  philosophy.
- Motion enabled. Do not globally disable transitions to make snapshots stable; handle that
  in the visual-regression config instead.

### Publish it

A local Storybook helps nobody outside the team. Publish it (Chromatic, static hosting, or
internal) and treat that URL as the system's address. Storybook emits an `index.json`
listing every story and generates prop metadata from your types, which is what makes it
machine-readable downstream.

---

## 4. The generated skill file

When the system is built, generate **one skill file** that lets a designer or an agent use
it without rediscovering it. One file, not a folder.

### The generation rule that matters

**Split it into generated and hand-written parts, and mark the boundary in the file.**

- **Generated from the system**: the token inventory, the component allowlist with import
  paths and variants, the scales. Emitted by a script from the token source and the
  registry, so it physically cannot drift from what was built.
- **Hand-written**: the visual philosophy, judgment calls a linter cannot make, known traps.
  This is the taste layer and no generator can produce it.

Regenerating must be a single command, and it must run whenever a component or token
changes.

### Required contents

1. **Where the system lives.** The published Storybook URL, stated as the source of truth.
2. **The hard rules** (below).
3. **The component allowlist**: every component, its import path, its variants. Generated.
4. **The token inventory**: semantic tokens by group, plus the spacing, radius, and type
   scales. Generated.
5. **The visual philosophy**: the three to five falsifiable sentences from discovery.
6. **Judgment calls**: Badge vs Chip, when a surface gets a border, density by surface type,
   where delight is allowed and where it is banned.
7. **Known traps** specific to this codebase.
8. **How to add something**, so a genuine gap becomes a contribution rather than an
   improvisation.

### The hard rules to state verbatim

These are the rules the user asked for, and they belong at the top of the generated file
where a model reliably reads them:

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
suggestion a model can weigh against other considerations. "If it is not a token, it does
not exist" is not.

### How design tools and agents fetch from it

Follow the same always-on plus on-demand split used everywhere else in this skill:

- **Always-on**: the generated skill file itself, carrying the allowlist and token
  inventory inline. Small, cheap, present before the model writes anything. Foundations
  cannot be fetched on demand because the model needs them before it knows it needs them.
- **On-demand**: Storybook for the detail. Its published `index.json` enumerates every
  story, and autodocs metadata carries prop types and descriptions. Point a Storybook MCP
  server or a fetch tool at that for full component APIs.

Keep the skill file small enough to always load. Push everything else behind the fetch.

### Also ship it as `AGENTS.md`

Symlink or copy the generated file to `AGENTS.md` at the repo root so coding agents pick it
up automatically without configuration. Same content, conventional filename.
