# Delivery: preview, approval, Storybook, and the generated skill file

The order is deliberate and non-negotiable:

```
foundations + tokens
  -> ONE HTML FILE, complete inventory, atomic layers, fully interactive
  -> USER APPROVAL
  -> React components
  -> Storybook
  -> generated skill file
```

**The HTML comes before React, and it contains everything.** Two separate reasons, and
both matter:

1. **Completeness.** The HTML is where you prove nothing is missing. Components get
   silently omitted when the inventory only exists in someone's head. If it is not in the
   HTML, it will be forgotten.
2. **Cost of rejection.** Storybook is config, addons, a build pipeline, and a story file
   per component. React components are typed, tested, and registered. Doing either before
   anyone agrees the system looks right means rebuilding it. One HTML file is cheap to
   produce and cheap to throw away.

---

## 1. The HTML preview

### What it is

**One self-contained `.html` file** a stakeholder opens by double-clicking. No server, no
install, no framework on their machine. It is simultaneously the approval artifact and the
completeness check.

### Structure: label every section by atomic layer

The file must be sectioned in this order, each section labelled with its layer name and a
count, so a reviewer can see at a glance what exists.

**A visible checklist at the top**, listing every component in the inventory with a tick,
so anything missing is obvious without scrolling.

#### FOUNDATIONS (rendered, not described)

Not components, but they come first because everything below is built from them.

- Color: every ramp, plus semantic token swatches with names and values, in both themes
- Typography: every type role shown in situ, not as a list of sizes
- Spacing scale, radius scale, border widths
- Elevation and shadow, in both themes
- Motion: live samples per duration and easing token, actually moving
- Iconography: the grid, every size, and the icon set
- Breakpoints, focus ring, opacity

#### ATOMS (24)

Button · Icon Button · Link · Text · Heading · Icon · Badge · Status Dot · Avatar ·
Divider · Spinner · Skeleton · Checkbox · Radio · **Switch** · **Toggle Button** ·
Text Input · Text Area · Number Input · Select · Slider · Kbd · Code · Progress Bar

#### MOLECULES (27)

Field (label + input + help + error) · Button Group · **Toggle Button Group** ·
Segmented Control · Avatar Group · Breadcrumbs · Pagination · **Tab List / Tab Group** ·
**Accordion (Collapsible)** · Card · Clickable Card · Selectable Card · Alert · Banner ·
Toast · Tooltip · Popover · Hover Card · Dropdown Menu · More Menu (overflow) ·
Search Input · Stepper · Empty State · Chip / Tag · Date Input · Timestamp · Metadata List

#### ORGANISMS (16)

App Shell · Top Nav · Side Nav · **Side Drawer** · **Bottom Sheet** · Dialog / Modal ·
Command Palette · Table · Data Table · List · Tree List · Toolbar · Carousel ·
Calendar / Date Picker · File Upload · Typeahead / Combobox

#### LAYOUT PRIMITIVES (6)

Stack · Grid · Section · Container · Aspect Ratio · Resize Handle

#### PAGES (2-3)

Fully composed real screens assembled from everything above. Components in isolation
always look fine. A composed screen is where slot collisions, density mistakes, and
genuinely missing components become visible.

### The tier-3 exemption trap, closed explicitly

`components.md` correctly warns that Data Table, Date Picker, Combobox, File Upload,
Charts, and Rich Text Editor are person-month time sinks. **That guidance is about React
build ORDER. It is not permission to omit them from the HTML.**

The distinction, stated so an agent cannot misread it:

| Artifact | Question it answers |
|---|---|
| HTML preview | What will exist? **Completeness.** Everything appears. |
| React build order | What gets built first? **Sequencing.** Tiers apply here. |

A deferred component still appears in the HTML, so the user approves the full scope and
nobody discovers a missing Tab Group three weeks later.

### Hard requirements

- **Single file.** CSS in a `<style>` block, JS in a `<script>` block. Fonts and logo may
  be linked or embedded as data URIs. If it needs a second file, it is wrong.
- **Fully interactive, not a screenshot sheet.** Tabs switch. Accordions expand. Switches
  toggle. Drawers slide in and trap focus. Dropdowns close on escape. Modals overlay.
  Steppers increment. Sliders drag. If a component has behavior, the behavior works.
- **Motion running** on the real motion tokens. This is the only artifact where the user
  can feel the motion personality before committing to it, which is why motion is a
  discovery question.
- **Every variant and every state**, not just defaults. Rest, hover, pressed, focus,
  disabled, error, loading, empty, selected. Interaction-only states shown statically side
  by side too, so nothing has to be hunted for.
- **Working theme toggle**, light and dark live. Plus any brand or density axis.
- **Real content.** Real product nouns, realistic string lengths, plausible numbers. Lorem
  ipsum hides every layout problem that matters.
- **The company logo in place**, in the header and wherever the brand actually appears. A
  system reviewed without it gets judged as a generic kit rather than as their product.

### Why plain HTML rather than a quick Storybook

It renders anywhere, including on a phone or in a Slack message. It survives being emailed
to someone without Node. It costs an hour instead of a day. And it forces the components to
work as plain DOM and CSS before a framework abstraction hides a problem.

---

## 2. Where the inventory comes from

The list above is a **floor, not a ceiling.** It is derived from two sources:

**Meta's Astryx** ([astryx.atmeta.com/components](https://astryx.atmeta.com/components)),
a real shipping system with roughly 90 components across Action, Chat, Container, Content,
Data Input, Feedback and Status, Layout, Navigation, Overlay, Table and List, and Utility.

**Component prevalence data** across published systems (see `components.md`).

Two things worth noticing about Astryx:

- It categorizes by **function** (Action, Overlay, Data Input) while atomic layers organize
  by **composition**. Both are valid and they are not in conflict. Use functional grouping
  for browsing and docs, atomic layers for build order and dependency direction.
- It ships an entire **Chat cluster** (Chat Composer, Chat Message, Chat Tool Calls,
  Chat System Message) because Meta builds AI products. That is direct evidence for
  inventory-driven building: the canonical list is the minimum, and domain components go on
  top of it. A payments company ships an `Amount`; a health product ships a vitals display.

Add what your interface inventory found. Never subtract from the floor without saying so
and getting agreement.

---

## 3. The approval gate

Present the preview and ask for a decision, not for vague feedback:

1. Does this feel like your product? If not, which part: color, type, density, radius, or
   motion?
2. Is the motion personality right: too flat, too much, or about right?
3. **Is anything missing from the inventory, or too thin to carry your real content?**
4. Anything here you would not use?

**Only after explicit approval does React work begin.** If the answer is "close, but the
density is wrong," fix it in the HTML and re-present. Iterating one HTML file is cheap.
Iterating 70 React components is not.

Record the approval. It is the point at which the visual language and the scope both stop
being negotiable and become a contract.

---

## 4. Storybook implementation

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

## 5. The generated skill file

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

Keep the skill file small enough to always load. Push everything else behind the fetch.

### Also ship it as `AGENTS.md`

Symlink or copy the generated file to `AGENTS.md` at the repo root so coding agents pick it
up automatically without configuration. Same content, conventional filename.
