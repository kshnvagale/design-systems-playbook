# Visual language: the craft layer

Tokens, naming, and enforcement decide whether a system is *correct*. They do not decide
whether it is any good to look at. This file covers the decisions that produce the look
and feel, and it exists because a system built only from the other reference files will
be architecturally sound and visually generic.

The useful framing is Diana Wolosin's, from building a design-system MCP at Indeed: some
of what a system encodes is **correctness** (right component, spacing follows the rules),
which is mechanically checkable, and some is **judgment** (does this breathe, does it feel
like this product), which has to be made specific before it can be encoded at all. Do not
skip the second because only the first is testable.

**Every section here ends with the implementable artifact.** A design decision that does
not land as a token, a prop, a rule, or a documented constraint is a preference, not a
system decision, and it will not survive contact with a developer or an agent.

---

## Typography

### Decisions to make

**How many type roles?** Not how many sizes. A role is a named, complete text style with
a purpose (`display`, `headline`, `title`, `body`, `bodyEmphasized`, `label`, `caption`).
Most products need 8-12. Material 3 ships around 15 across five families. Fewer than 6 and
designers will start overriding; more than about 15 and nobody can remember which is
which.

**Scale ratio.** Pick a ratio and derive sizes rather than choosing them by eye. Common
choices: 1.125 (major second, dense UI), 1.2 (minor third, balanced), 1.25 (major third,
more contrast), 1.333 and up (editorial, marketing). Dense product UI usually wants
1.125-1.2 because large ratios waste vertical space and force awkward jumps.

**Round to whole pixels.** A pure geometric scale produces 17.28px. Round it. The ratio is
a generator, not a constraint you must honor exactly.

**Line height by role, not globally.** Tight for large text (1.1-1.25 on display and
headline, because large type needs less leading to feel connected) and looser for body
(1.4-1.6). A single global line height makes headlines feel airy and body feel cramped.

**Line length.** The long-standing typographic guidance is roughly 45-75 characters per
line for comfortable reading. This is a **layout** constraint, not a type constraint, so
it becomes a max-width on text containers, not a font-size decision.

**Weight strategy.** Decide which weights are real. Most products need two or three (400,
600, and sometimes 700). Every extra weight is a font file and a decision nobody will make
consistently. Then decide the emphasis rule: **weight or color, pick one as primary.**
Using both for the same purpose produces mush.

### The implementable artifact

Composite type tokens, one per role, bundling family, size, weight, line height, and
letter spacing. Not four separate atomic tokens a designer has to remember to combine.

```
type.body            = { family, 14px, 400, 20px, 0 }
type.bodyEmphasized  = { family, 14px, 600, 20px, 0 }
```

Plus a documented rule such as: *emphasis is carried by weight; secondary and tertiary
text color express hierarchy, not importance; never recolor text to make it stand out.*

---

## Layout, grid, and responsive

### Decisions to make

**Does this product need a column grid at all?** Marketing pages and content sites do.
Dense application UI often does not, and a grid imposed on a data table is decoration.
Decide honestly rather than adding one because design systems are supposed to have one.

**Breakpoints.** Derive from content, not from device names. Devices change; the point at
which your layout breaks does not. IBM Carbon's [2x Grid](https://carbondesignsystem.com/elements/2x-grid/overview)
is the clearest published reference and worth copying outright if you have no reason not
to:

| Breakpoint | Width | Columns | Margin |
|---|---|---|---|
| Small | 320px | 4 | 0 |
| Medium | 672px | 8 | 16px |
| Large | 1056px | 16 | 16px |
| Max | 1584px | 16 | 24px |

Carbon's underlying idea is worth understanding even if you pick different numbers: an
**8px mini unit** as the basic geometric unit, with columns, rows, boxes, margins, and
padding all composed from multiples of it, and the grid built by dividing or multiplying
by two. That is what makes the rhythm hold together rather than being a set of arbitrary
widths.

**Container max-widths.** Full-bleed dashboards and constrained content pages are
different modes. Name them (`container.content`, `container.wide`, `container.full`)
rather than hardcoding a number per page.

**Density.** The single most under-decided dimension. Decide it per surface type, not
globally:
- Scanning many items (grids, tables, lists): tight, 8-12px inside, 12-16px between.
- Reading and deciding about one thing (forms, settings, detail views): roomy, 24px
  between fields.

If you cannot decide, ask which of those the user is doing. That question resolves it
almost every time.

### The implementable artifact

Breakpoint tokens, column and gutter tokens, named container widths, and a documented
density rule per surface type. Layout width is one of the few places where arbitrary
values are legitimate, so scope your lint rules to allow it (see `ai-agents.md`).

---

## Iconography

Consistently the most under-specified part of a new design system, and the one that drifts
fastest because icons get added one at a time by whoever needs one.

### Decisions to make

**Grid size.** 24x24 is the contemporary default. 16x16 for data-dense dashboards, 32x32
for touch-first or large interfaces. Pick one primary grid and derive the rest.

**Keylines.** Material's [system icon spec](https://m2.material.io/design/iconography/system-icons.html)
defines keyline shapes inside the 24dp grid (square 18dp, plus circle and rectangle
keylines) so that different icon shapes read as the same optical size. Without keylines,
a circular icon looks smaller than a square one at identical dimensions.

**Stroke.** Fix a single weight and stick to it. Material uses 2dp on a 24dp grid with
squared terminals. Also decide corner radius and how strokes behave on diagonals. Most
systems fail here specifically because these rules were never written down.

**Style language.** Outlined, filled, two-tone, or duotone. Pick one lane. If you need
both outlined and filled, define what the pairing *means* (commonly: filled equals active
or selected, outlined equals default) rather than leaving it to taste.

**Optical sizing over mathematical sizing.** Circles, triangles, and squares of identical
bounding-box size do not read as the same weight. Adjust so they look equal, which means
they will not measure equal.

**Sizes.** Two or three, tied to type roles so an icon beside a label matches its cap
height. `icon.sm` next to `type.body`, `icon.md` next to `type.title`.

### Icon usage rules worth writing down

- Icons alone are ambiguous. Anything not universally understood (a magnifier, an X, a
  hamburger) needs a visible label or an accessible name.
- Decorative icons get `aria-hidden`; meaningful icons need an accessible label. This is
  the single most common accessibility bug in icon usage.
- One metaphor, one meaning, system-wide. If a checkmark means "verified" in one place, it
  cannot mean "completed" in another.

### The implementable artifact

An icon component with a fixed size scale, a production checklist every new icon must
pass (aligns to grid, follows stroke rules, optically balanced, legible at the smallest
size, uses an approved metaphor), and an accessible-name prop that is required unless the
icon is explicitly marked decorative.

---

## Motion

### Write motion principles first, and make them falsifiable

Adobe's guidance from building Spectrum's motion layer gives the test: **a strong motion
principle is one where you can easily state the opposite.** "Motion should not be
distracting" passes, because distracting motion is a real thing you can point at. "All
motion should be beautiful" fails, because it rules nothing out and cannot settle an
argument. Spectrum's own are: purposeful, intuitive, seamless.

Material's three are informative, focused, and expressive, applied across four uses:
hierarchy, feedback, status, and character animation. That second list is the more useful
artifact, because it tells you *where* motion belongs.

### Pick a motion personality, explicitly

Every serious system now offers two modes, and the split is real rather than cosmetic:

| | Utilitarian | Expressive |
|---|---|---|
| Carbon | `productive` easing | `expressive` easing |
| Material 3 | `standard` scheme, eases into final values | `expressive` scheme, overshoots to add bounce |
| Use for | Dense working surfaces, data, forms, anything repeated hundreds of times a day | Hero moments, onboarding, celebration, key interactions |

Carbon ships different cubic-bezier curves per mode, which is the implementable version of
this idea:

| Curve | Productive | Expressive |
|---|---|---|
| Standard | `cubic-bezier(0.2, 0, 0.38, 0.9)` | `cubic-bezier(0.4, 0.14, 0.3, 1)` |
| Entrance | `cubic-bezier(0, 0, 0.38, 0.9)` | `cubic-bezier(0, 0, 0.3, 1)` |
| Exit | `cubic-bezier(0.2, 0, 1, 0.9)` | `cubic-bezier(0.4, 0.14, 1, 1)` |

Most products want productive as the default with a small, named set of expressive
moments. Decide which moments those are rather than leaving it to whoever builds the
screen.

Worth knowing: Material 3 Expressive (2025) replaced its easing-and-duration model with a
**physics/spring system**, on the reasoning that springs handle gestures, interruptions,
and retargeting more naturally than fixed curves. If your product is gesture-heavy or
native, springs are worth evaluating over duration tokens.

### Decisions to make

**What earns motion.** Motion should clarify a relationship: where something came from,
what changed, what is now loading. Motion that only decorates is cost without benefit.

**Duration by purpose**, following Material's buckets:
- 50-200ms for hover, press, and small feedback
- 250-400ms for on-screen transitions
- 450-600ms for large expressive transitions
- Above 700ms is rare and usually a mistake in product UI

**Easing carries meaning.** Things entering the screen decelerate; things leaving
accelerate. Symmetric easing on both makes the interface feel mechanical.

**Reduced motion is a requirement, not a nicety.** Every motion token needs a
`prefers-reduced-motion` answer, and it is usually "no movement, keep the opacity change,"
not "no transition at all." A softer option worth offering: keep motion at the container
level and drop it on individual elements, rather than removing it entirely.

**Choreography**, meaning multiple elements moving during one transition, is what separates
a system that feels designed from one that merely animates. Two rules carry most of the
value:

- **Entrances inform exits.** If a panel slides in from the right, it leaves to the right.
  Asymmetry here reads as a bug even when users cannot articulate why.
- **Constrain motion paths.** Carbon's rule is that motion paths trace lines along the
  grid and never run diagonally. Any equivalent constraint works; having none is what
  produces chaos.

### Carbon's evaluation checklist, worth stealing

Before shipping a motion, check: does it respond to user input, do micro-interactions fall
within roughly 90-120ms, do large transitions include a continuous element that carries
the user across, and are the easing curves the right ones for the mode.

### The implementable artifact

Motion tokens named by intent rather than value, which is Atlassian's model:
`motion.popup.enter` bundling duration, easing, and property. Consumers reference a
decision, so you can retune the milliseconds without touching component code. Plus a
documented reduced-motion fallback per token.

---

## Delight, and where it is banned

Delight is a system concern, not a garnish added by whoever is feeling creative. Left
undecided it either never appears or appears in exactly the wrong place.

### Decisions to make

**Where delight is allowed.** It lives in a small, predictable set of places: first run
and onboarding, empty states, success and completion, and small micro-interaction
feedback. Naming them makes delight a deliberate act rather than a mood.

**Where it is banned.** More important than the allowlist, and non-negotiable in finance,
health, and anything legally consequential:

- Never on an error, failure, or decline.
- Never when someone's money, health, or legal standing has gone wrong.
- Never on a destructive confirmation.

A celebratory animation on a failed payment is not a small mistake. It reads as the
product not understanding what just happened to the user.

**Proportionality.** Feedback weight should match stakes. A destructive action warrants
more friction and more confirmation than a reversible one. Both mismatches erode trust: a
subtle animation on a high-stakes action, and an intrusive one on a trivial click.

**The restraint test.** Dan Saffer, who named microinteractions, frames them as "an
exercise in restraint, in doing as much as possible with as little as possible." The
practical version, applied before adding any: **name the uncertainty this moment creates
for the user.** If you cannot name it, you are decorating.

### Anatomy, for documenting one properly

Saffer's four parts are the right structure for a microinteraction spec: **trigger** (what
starts it), **rules** (what happens), **feedback** (how the user knows), and **loops and
modes** (what happens on repeat or in a different state). A microinteraction documented
without its rules and modes will be rebuilt inconsistently.

### The implementable artifact

A named list of delight moments with the component each belongs to, an explicit banned
list in the content and motion guidance, and expressive motion tokens that exist
separately from productive ones so using the wrong one is a visible choice rather than an
accident.

---

## Illustration and imagery

Usually undecided, and then invented per team, which is one of the fastest ways a single
product starts looking like three.

### Decisions to make

- **Medium**: photography, illustration, 3D, or abstract geometry. Pick one primary.
- **Where it appears**: empty states, onboarding, error pages, marketing surfaces. These
  are the recurring slots.
- **Whether it carries meaning or is decorative.** This has an accessibility consequence:
  decorative illustration should be skipped by screen readers, and no information may
  live only in the image.
- **Where the brand is loud versus invisible.** Most strong products are visually quiet in
  the working surfaces and expressive at the edges. State the boundary.

### The implementable artifact

A named set of illustration slots tied to components, sizing rules, and a default
`aria-hidden` on decorative imagery so the accessible behavior is the path of least
resistance.

---

## Sound and haptics

Only relevant for native and mobile, and routinely forgotten until an engineer asks. If in
scope, they are token categories like any other (a small set of named feedback events, not
per-screen decisions). If out of scope, record that as an explicit decision rather than an
oversight.

---

## State design

Empty, loading, and error states are where systems are judged and where they are usually
thinnest. Treat them as designed patterns, not as components someone will improvise.

### The three kinds of empty state

Carbon's [empty states pattern](https://carbondesignsystem.com/patterns/empty-states-pattern)
is the clearest breakdown, and the three are genuinely different problems:

| Type | When | What the copy must do |
|---|---|---|
| **First use** | Day zero, nothing exists yet | Explain what will appear here and give the action that creates the first one |
| **Action result** | User filtered or searched, nothing matched | Say what was searched, offer a way to widen or clear it |
| **Error** | Something failed to load | Say what happened in plain language, no error codes, and what to do next |

Layout guidance worth adopting: keep elements left-aligned as a block rather than centered,
except inside small tiles. Centered empty states in a dense app read as a broken page.

**Accessibility note that gets missed:** empty-state illustrations are decorative and
should be skipped by screen readers. Never put information only in the illustration.

### Loading

Decide the rule once: skeletons for content whose shape you know, spinners for actions
whose duration you do not. Skeletons that do not match the eventual layout are worse than
no skeleton, because the page visibly jumps.

### Errors

Nielsen's rule is the standard and still correct: express the problem in plain language,
no codes, indicate precisely what went wrong, and constructively suggest a solution.

### The implementable artifact

An `EmptyState` component with variants for the three types, a documented loading rule,
and error message templates. These belong in the system precisely because product teams
under time pressure will otherwise ship a bare "No data."

---

## Content design

Most engineering-led design systems skip this entirely. Polaris, Carbon, and PatternFly
all treat it as a first-class pillar, and it is the cheapest way to make a product feel
coherent.

### Voice versus tone

**Voice is constant. Tone changes with context.** Same product, same voice, but the tone
of a celebratory confirmation and a failed-payment error are not the same.

### How to define voice usefully

Do not write adjectives. PatternFly's method is better and takes an afternoon: pick four
or five voice traits, each phrased as a **tension** so it rules something out, then give
each a Don't/Do table.

Their traits are shaped like: *helpful but humble, not arrogant. Authentic but adaptable,
not stubborn. Open but ordered, not chaotic.* The "not X" is what makes it usable, because
it lets a writer or a model reject a candidate sentence.

### Microcopy rules to standardize

These recur in every system and are worth deciding once:

- **Sentence case or title case** for headings, buttons, labels. Pick one. Sentence case
  is the modern default and is easier to get right.
- **Buttons say what happens**, not "OK" or "Submit." "Save changes," "Delete list."
- **Person and pronouns.** Is it "your account" or "my account"? Does the product say
  "we"? Decide, because inconsistency here is very visible.
- **Contractions.** Usually yes, they make copy sound human.
- **Error messages** never blame the user, never expose a code, and always offer the next
  step.
- **Dates, numbers, and currency formatting**, which is a real system decision in fintech,
  data, and international products.

### The implementable artifact

A content section in the docs, per-component content guidance on component pages (how to
word this modal's title, this button's label), and the voice traits with Don't/Do tables.
For an agent-consumed system, this belongs in the always-on foundations layer, because
copy decisions happen while the model is writing the component, not afterward.

---

## Keeping it dev-friendly

The craft decisions above are only useful if they arrive in a form someone can build
against. Four rules that keep the design and engineering halves joined:

**1. Every visual decision names its token.** "Cards feel a bit tight" is not actionable.
"Card padding moves from `space.12` to `space.16`" is a one-line change with a clear blast
radius.

**2. Design and code use identical vocabulary.** Same component names, same variant names,
same casing, no translation layer. This is Blade's explicit rule and it is also what makes
the system legible to AI codegen.

**3. If a design cannot be expressed in existing tokens and props, that is a system gap,
not an override.** The correct response is to add a variant, not to let the instance
diverge. A designer requesting a value that is not on the scale is reporting a missing
token, and should be treated as reporting a bug rather than being told no.

**4. State the constraint, not just the intent.** "Generous whitespace" is unbuildable.
"24px between form fields, 16px between grid cards, 12px inside a card" is buildable, and
it survives being handed to an agent.

The test for any craft guidance you write: **could a developer or an agent implement this
without asking a follow-up question?** If not, it is not finished.
