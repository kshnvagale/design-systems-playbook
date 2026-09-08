# Delivery: preview, approval, Storybook, and the generated skill file

The order is deliberate and non-negotiable:

```
foundations + tokens
  -> PREVIEW 1: the base system (no page templates)
  -> APPROVAL GATE 1: is the system right?
  -> ASK: do you want page templates? which ones? collect input
  -> PREVIEW 2: page templates, built only from approved components
  -> APPROVAL GATE 2: are the templates right?
  -> ASK: are marketing surfaces in scope? (see section 3b)
  -> PREVIEW 3: marketing surfaces, only if they are in scope
  -> APPROVAL GATE 3: are the surfaces right?
  -> foundation choice -> React -> Storybook -> generated skill file
```

**Two gates for the product system, plus a third only if marketing surfaces are in
scope.** Page templates are a separate concern from the system itself. A team that ran
this with a single gate got templates built in the same pass as the system, before anyone
had said which pages they wanted or supplied a single screenshot. Marketing surfaces are a
third concern again, with their own preview and their own gate when they are in scope, and
skipped entirely when they are not. See `marketing-surfaces.md` for what preview 3
contains, and section 3b below for the question that decides whether it happens.

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
Segmented Control · Avatar Group · Breadcrumbs · Pagination · **Tabs** ·
**Accordion** · Card · Clickable Card · Selectable Card · Alert · Banner ·
Toast · Tooltip · Popover · Hover Card · Menu · More Menu (overflow) ·
Search Input · Stepper · Empty State · Chip · Date Input · Timestamp · Metadata List

#### ORGANISMS (16)

App Shell · Top Nav · Side Nav · **Side Drawer** · **Bottom Sheet** · Dialog ·
Command Palette · Table · Data Table · List · Tree List · Toolbar · Carousel ·
DatePicker · File Upload · Combobox

#### LAYOUT PRIMITIVES (6)

Stack · Grid · Section · Container · Aspect Ratio · Resize Handle

#### ONE COMPOSED SCREEN (not templates)

Exactly one realistic assembled screen, as a **sanity check on the system**, not as a
template deliverable. Components in isolation always look fine; one composed screen is
where slot collisions, density mistakes, and genuinely missing components become visible.

**Page templates are not part of preview 1.** They come after gate 1, with their own
input and their own gate. See below.

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

### Layout: fixed left nav, scrolling right pane

**Do not build this as one long scrolling column.** A complete inventory is roughly 73
components plus foundations plus one composed screen, which lands somewhere around 60,000px of
vertical scroll. At that length the only way to find anything is to scroll past everything
else, and a reviewer cannot tell what exists without seeing all of it.

Use a two-pane docs shell:

```
┌──────────────────┬────────────────────────────────────────┐
│  NAV (fixed)     │  PREVIEW (scrolls)                      │
│  240-280px       │                                         │
│                  │  ┌───────────────────────────────────┐  │
│  Foundations     │  │ Button                            │  │
│  Atoms      24   │  │ variants, sizes, all states       │  │
│   Button       ● │  └───────────────────────────────────┘  │
│   Icon Button    │                                         │
│   Link           │  ┌───────────────────────────────────┐  │
│  Molecules  27   │  │ Icon Button                       │  │
│  Organisms  16   │  └───────────────────────────────────┘  │
│  Layout      6   │                                         │
│  Pages           │                                         │
└──────────────────┴────────────────────────────────────────┘
```

**Left pane requirements:**

- Fixed or sticky, full height, independently scrollable.
- Grouped by atomic layer, in the same order as the sections, with a count per group.
- Every component listed by name. This doubles as the completeness checklist: a reviewer
  scans the nav rather than the page to see what exists.
- Active item tracks scroll position, so you always know where you are.
- A filter or search box. Seventy-three items is too many to scan reliably.
- A theme toggle and any other axis switch, pinned so it is reachable from anywhere.

**Right pane requirements:**

- Scrolls independently of the nav.
- **Constrain content width** (roughly 1100-1200px) rather than letting it stretch to the
  viewport. Full-bleed only for layout primitives and the composed screen, where the width is
  the point.
- One component per block, each block visually separated as its own panel or card with a
  heading, not run together.

### Spacing: let it breathe

The most common failure in a generated preview is everything crammed together, which makes
it impossible to tell where one component ends and the next begins.

| Between | Space |
|---|---|
| Major sections (Atoms, Molecules) | 96-128px, with a visible divider or heading band |
| Component blocks within a section | 64-80px |
| Variant rows inside one component | 32-40px |
| Individual specimens in a row | 16-24px |
| A component and its label or caption | 12-16px |

Give every component block a generous internal padding (32-40px) and a subtle border or
surface step so it reads as a discrete unit. Label every specimen with its variant and
state name. An unlabeled grid of buttons is decoration; a labeled one is documentation.

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
- **Correct ARIA roles on styled native controls.** A checkbox styled as a switch still
  announces as a checkbox unless it carries `role="switch"`. Same for anything where the
  visual affordance and the native element disagree. This is the single most common
  accessibility bug in a generated preview, and it survives into React if you do not catch
  it here.
- **The company logo in place**, in the header and wherever the brand actually appears. A
  system reviewed without it gets judged as a generic kit rather than as their product.

### Build it with a fan-out, not serially

This file holds ~73 components plus foundations plus one composed screen. That is far too much
for one agent in one pass, and quality visibly degrades toward the end of a long serial
build. **Dispatch subagents.** The mechanics, because a single file cannot use normal
exclusive-file ownership:

1. **The orchestrator writes the shell first, alone.** The `<style>` block with every token
   as a CSS custom property in both themes, the theme-toggle script, shared base and
   utility classes, the section scaffolding, and the checklist. **This shell is the
   contract every agent builds against.**
2. **Fan out by section.** Each agent returns an **HTML fragment**: markup plus any
   component-specific script for its section. Agents never write the file.
3. **The orchestrator concatenates and verifies.** No agent touches the target file. That
   is the only reason a single-file fan-out is safe.

A workable split:

| Agent | Section |
|---|---|
| 1 | Foundations |
| 2 | Atoms A-L |
| 3 | Atoms M-Z |
| 4 | Molecules A-M |
| 5 | Molecules N-Z |
| 6 | Organisms |
| 7 | Layout primitives and the one composed screen |
| **Orchestrator** | Shell, stitch, verification |

**Every agent gets the same inline spec**: the exact token names available (they may not
invent any), the class conventions from the shell, the required states per component, and
the instruction that motion runs on real tokens.

**After stitching, verify**: every checklist item present, theme toggle working across all
sections, no invented tokens or colors, interactions actually functional rather than static
markup.

### Solving inconsistency properly: agents write markup, never CSS

"Make the shell thick" is a hedge. The actual fix is the same one used everywhere else in
this skill: **make invention structurally impossible rather than discouraged.**

**The shell must define every component base class, not just the tokens.** Not
`--color-action-primary` alone, but `.btn`, `.btn--primary`, `.btn--sm`, `.input`,
`.input--invalid`, `.card`, `.chip`. Then the rule for every agent is one line:

> Return markup only. You may not write CSS, `<style>` blocks, or inline `style`
> attributes. Compose from the classes defined in the shell. If you need something the
> shell does not provide, STOP and report it as a gap. Do not create it.

An agent that cannot author a style cannot invent one. Inconsistency stops being a matter
of discipline and becomes structurally unavailable, and the only remaining move when
something is missing is the one you want: report the gap.

### Verify it mechanically, not by eye

Because the vocabulary is closed, the check is deterministic:

```
1. Extract every class attribute value in the stitched file.
2. Diff that set against the class names the shell defines.
3. Any class not in the shell is invention, and the diff names it exactly.
```

Plus three greps that must all return zero outside the shell:

- `<style` blocks
- inline `style=` attributes
- hex, `rgb()`, or `hsl()` literals

That is seconds of work against 73 components, versus eyeballing them.

### Sequence: shell, then one reference section, then fan out

Do not go straight from shell to fan-out. Build **one section yourself first**, completely,
and pass it to every agent as a worked example.

Models weight a nearby concrete example far more heavily than stated instructions. One
fully-built, correctly-styled section in the prompt does more than any amount of prose
about conventions. It also proves the shell is actually sufficient before seven agents
discover it is not.

This is the general rule from `orchestration.md` ("build the first instance of a new
pattern yourself, then fan out") applied to the preview.

### What remains, honestly

This closes styling invention. It does not close **structural** inconsistency: two agents
can compose valid classes into differently-shaped markup for similar components. The
reference section is the main defense, and a final visual pass over the stitched file is
the backstop. Gates prove discipline, not sufficiency, which is the same limit described in
`evaluation.md`.

### Why plain HTML rather than a quick Storybook

It renders anywhere, including on a phone or in a Slack message. It survives being emailed
to someone without Node. It costs an hour instead of a day. And it forces the components to
work as plain DOM and CSS before a framework abstraction hides a problem.

---

## 2. Where the inventory comes from

The list above is a **review instrument**, not a build order. `components.md` is the single
source and explains the review procedure. It is derived from two sources:

**Meta's Astryx** and **component prevalence data** across published systems. `components.md`
carries both, including Astryx's category list and the two transferable lessons about how it
is organized. Do not restate them here; this file only needs the consequence.

The consequence for the preview: functional grouping is for browsing and docs, atomic layers
are for build order and dependency direction, and the preview sections use the atomic layers.
Astryx's own inventory is also the clearest evidence for inventory-driven building, since it
ships a whole Chat cluster because Meta builds AI products. Review the canonical list item by
item, record a reason for anything you exclude, and add domain components on top of whatever
you include. A payments company ships an `Amount`; a health product ships a vitals display.

Add what your interface inventory found. **Subtracting is allowed and often correct** -
review the list item by item and record one line for anything you exclude. See
`components.md`, which is the single source for the inventory and explains the review
procedure. A ten-component internal tool should end up excluding most of this list, with
reasons.

---

## 3. Approval gate 1: the base system

Present the preview and ask for a decision, not for vague feedback:

1. Does this feel like your product? If not, which part: color, type, density, radius, or
   motion?
2. Is the motion personality right: too flat, too much, or about right?
3. **Is anything missing from the inventory, or too thin to carry your real content?**
4. Anything here you would not use?

**Only after explicit approval does React work begin.** If the answer is "close, but the
density is wrong," fix it in the HTML and re-present. Iterating one HTML file is cheap.
Iterating 70 React components is not.

Add one more, because this is the last cheap moment for it:

5. **Are the component names right?** Renaming after React exists is a breaking change
   touching every consumer. Here it is a find and replace.

Record the approval. It is the point at which the visual language and the scope both stop
being negotiable and become a contract.

---

## 3a. Ask about page templates. Do not assume.

Once gate 1 passes, **ask**. Do not start building pages.

> The base system is approved. Do you want me to build page templates on top of it?
>
> If yes, I need three things: the list of pages or screens you want, any screenshots or
> references for each, and real content where you have it. Real product names, real copy
> lengths, real numbers. Templates built on placeholder content break the moment real
> content arrives.

**The answer may legitimately be no.** Some teams want the system and will compose pages
themselves. Building templates nobody asked for is exactly the failure this split exists to
prevent.

### Preview 2: templates only

Built strictly from approved components, in the same two-pane shell, same fan-out method.

**The hard rule: a template may not introduce a new component.** If a page needs something
the system does not have, that is a system gap to report and add deliberately, then
regenerate. It is not license to invent inside a template. This is the reuse-over-invention
rule applied one layer up, and it is the rule most likely to be broken under deadline,
because a template feels like a one-off.

### Approval gate 2: the templates

Different questions from gate 1. Ask these:

1. Does this page do its job? What is the one action you want a user to take here?
2. Is the content hierarchy right? Is the most important thing the most prominent thing?
3. Is anything missing from the flow, as opposed to missing from the system?
4. Does it hold up with your real content lengths, not the placeholder ones?

**Why the ordering matters:** templates built after approval get built once. Templates
built before it get rebuilt every time a component changes, which during a first build is
constantly.

---

## 3b. After gate 2: ask before writing any React

Gate 2 passing does not mean "start building." There are two more questions, and both
have real consequences.

> Templates approved. Two things before I write any React:
>
> **1. Do you want marketing surfaces too** (landing page, pricing, blog index), or is
> this product only? Marketing needs an extended type and spacing scale and its own
> component set, so it changes what I build, and a yes adds a third preview and a third
> gate before I am done.
>
> **2. What should I build the components on?** Three options, and I would recommend
> [X] for you because [reason from discovery]. Here are the tradeoffs.

The first question may already be answered from discovery. Ask it again anyway if it was
vague, because "we'll need a website at some point" and "the marketing site is in scope"
are different projects. See `marketing-surfaces.md`.

The second is the foundation choice below. **Do not pick it silently.** It determines what
the team maintains for years, and the tradeoffs are legible enough that the user should get
to weigh in.

Only after both are answered does React work start.

---

## 4. Choose the foundation before writing React

Put three options to the user. Do not pick silently. But do not present them as equally
weighted either: **there is a default, and you should say so.**

### The default: shadcn/ui as the base, Astryx as the reference

For most teams building a design system in 2026, **build on shadcn/ui and use Astryx as a
spec rather than a dependency.** Four reasons, in order of weight:

1. **Training data is the single biggest lever on agent codegen quality.** Models know
   shadcn cold and can recall its component APIs without a lookup. Astryx shipped
   recently and is in beta, so every one of its 150 component APIs has to be fetched from
   docs or MCP on every generation. That is a real, recurring context cost. "Designed for
   agents" is a claim about affordances; "the model already knows it" is a different and
   currently stronger property.
2. **You own the source from day one.** No beta dependency under your entire component
   layer, no upstream API churn, no waiting on someone else's release. Astryx's `swizzle`
   gets you there eventually; shadcn starts you there.
3. **shadcn is the registry pattern this skill already recommends.** Copy-paste
   distribution, a `registry.json` an agent can query, components you can lint against an
   allowlist. Adopting it makes the agent-consumability work in `ai-agents.md` cheaper,
   not harder.
4. **Adopting 150 components contradicts the first rule in this skill.** A design system
   is downstream of *your* product. Taking Meta's inventory wholesale is the exact
   opposite of building from your interface inventory, and most of those 150 will never
   be used.

**What to take from Astryx instead:** it is MIT, and its component inventory, API
conventions, and design conventions (published in the repo wiki) are genuinely excellent
and hard-won over eight years. Read them. Use the inventory as your completeness floor,
which `components.md` already does. Steal the conventions. Do not take the dependency.

**When Astryx-as-dependency is genuinely the right call:** a large team needing broad
surface area fast, where the widgets are explicitly not the differentiator, beta risk is
acceptable, and humans rather than agents write most of the code.

**When from-scratch is right:** the system itself is the product or a real competitive
asset, the visual language is distinctive enough that you would override most of a base
library anyway, or there are genuinely unusual interaction requirements.

Recommend against the discovery answers, and say which one you are recommending and why.

**The rule that survives all three:** the approved visual language does not change based on
this choice. Same tokens, same inventory, same philosophy. This decides what the components
are built **on**, not what they **are**.

### Option 1: From scratch on headless primitives

Radix Primitives or React Aria underneath, your own styling layer on top.

- **Best when** the system is a genuine differentiator, the visual language is distinctive,
  or there are unusual interaction requirements.
- **Cost:** highest. Every component is yours to build, test, and maintain.
- **Upside:** you own everything and nothing fights you.

### Option 2: Adopt Astryx and theme it

Meta's open source design system ([github.com/facebook/astryx](https://github.com/facebook/astryx),
MIT, currently in beta).

- **150+ accessible components.** Grew inside Meta over eight years, powers 13,000+ apps
  there.
- **React 19+, authored with StyleX, but no styling lock-in for consumers.** Override with
  `className` using Tailwind, CSS modules, or plain CSS.
- **Install:** `@astryxdesign/core`, a theme such as `@astryxdesign/theme-neutral`, plus
  `@stylexjs/stylex`. CLI is `@astryxdesign/cli`.
- **Theming is CSS custom property overrides**, so you can make it unmistakably yours
  without forking or wrapping component source. Ten themes ship with it (default, neutral,
  daily, butter, chocolate, matcha, stone, gothic, brutalist, y2k).
- **`swizzle` ejects a component's full source** into your project when you need to own it,
  so adopting is not a one-way door.
- Explicitly designed so people and AI assistants build the same way from the same
  reference.
- **Best when** you want breadth fast, accessibility handled, and your differentiation is
  in the product rather than the widgets.
- **Cost:** you inherit their component API and conventions. Beta status is a real risk to
  weigh.

### Option 3: Build on shadcn/ui

Copy-paste registry model on top of Radix.

- Component source is copied into your repo via CLI, so you own and can edit everything
  from day one.
- Enormous ecosystem familiarity, and **models are heavily trained on it**, which
  materially helps agent codegen.
- Trades easy upstream updates for full ownership.
- **Best when** the team already uses Tailwind, you want a fast start with total control,
  or agents write a large share of the code.

### Recommending one

| Their situation | Recommend |
|---|---|
| **Default, most teams** | **Option 3**, with Astryx read as a reference |
| Agents write a large share of the code | Option 3. Training-data familiarity dominates. |
| Already a Tailwind shop | Option 3 |
| The design system is itself the product or a competitive asset | Option 1 |
| Distinctive visual language you would override most of a base library for | Option 1 |
| Large team, broad surface fast, widgets not the differentiator, humans writing code | Option 2 |
| Regulated context needing audited accessibility fast | Option 2 or 3, both ship tested a11y |
| Multi-brand or white-label | Option 1 or 2, both theme cleanly through CSS custom properties |
| Beta dependency is unacceptable | Not Option 2 |

---

Once both gates have passed and the foundation is chosen, continue in `handoff.md`:
Storybook, the progress file, and the generated skill file.
