# Color palette construction and accessibility reference

## Decide how many hues. Then take the whole ramp.

Two separate decisions get conflated here, and getting them backwards is the most common
palette mistake.

- **How many hues do you generate and expose?** Be deliberate. This is where restraint
  belongs.
- **How deep is each hue that earns a ramp?** Full depth. Twelve steps. Do not economize
  here.

An earlier version of this file said status hues need only "3-4 steps each." **That was
wrong**, and worth correcting explicitly rather than quietly. Radix's own step map shows
why: a danger color realistically needs a subtle background (3), a hover (4), a border
(6-7), a solid fill (9), a solid hover (10), and text (11-12). That is 7-8 steps before
anything unusual happens. Ship a destructive Button with a hover state, plus an error
Alert, plus error text, and you have used most of the ramp.

**Truncating a ramp is worse than generating it in full.** The moment someone needs a step
that does not exist, they invent a value. That is precisely the failure this rule exists to
prevent, arrived at from the opposite direction.

### Where restraint actually belongs: hue count and exposure

**Availability is permission.** Designers and agents both reach for what is in front of
them. Generate and expose twelve hues when the product needs five, and the other seven will
appear somewhere, uncontrolled, then have to be policed forever. That is a real and
reliable failure pattern in a design team.

But notice what it is about: **how many hues are visible in the design surface**, not how
many steps each one has.

### Library versus generated: the distinction that resolves the argument

Radix explicitly says you may eventually want most of its scales, citing multiplayer color
assignment, per-task labelling, and status badges
([composing a palette](https://www.radix-ui.com/colors/docs/palette-composition/composing-a-palette)).
That reads as a contradiction of the restraint rule. It is not, because Radix and a
generated token set are different kinds of object.

| | Adopting Radix Colors | Generating your own |
|---|---|---|
| What it is | A library you import from | A token set you author, publish, and own |
| Unused scales live | In `node_modules` | In your token file, Figma, picker, and docs |
| Cost of the surplus | Zero, they are unavailable until imported | Real, every one is visible and pickable |
| Curation | ~31 scales, hand-tuned for harmony and accessibility | Whatever your generator emitted |

`import { blue, slate } from '@radix-ui/colors'` leaves the other scales invisible. They are
not in your design surface, so they are not an invitation.

**So:** if you are adopting Radix, import the scales you need, take all twelve steps of
each, and ignore the rest. There is no restraint problem to solve. If you are generating
your own, you own the exposure problem, so decide hue count deliberately.

Worth knowing if you adopt: Radix states its colors are **not intended to be customised**,
since customisation likely breaks the accessibility and harmony they were tuned for. Their
own recommendation is to use Radix for grays and semantic scales and add custom brand
scales alongside.

### Three palettes, one primitive foundation

The other half of the resolution. Radix bundles everything into one library because it is
one library. A design system should separate three palettes, because they answer to
genuinely different constraints.

| Palette | Hues | Depth | Theme-reactive | Optimized for |
|---|---|---|---|---|
| **UI** | brand + neutral + 3-4 status | Full 12-step ramps | Yes | State, hierarchy, interaction |
| **Categorical** | 8-12 | 1-2 steps each | Yes | Mutual distinguishability, colorblind safety |
| **Illustration** | Small locked set | Deep tonal range per hue | **No, frozen across modes** | Harmony within a composition |

**UI palette.** Everything this file otherwise discusses. Few hues, full ramps, driven by
the semantic token layer.

**Categorical palette.** This is what Radix's multiplayer and labelling examples actually
describe: colors used to tell things apart from each other, not to express state. User
avatars, task labels, chart series, tags. It needs breadth, and it has a hard constraint
the UI palette does not: **every hue must be distinguishable from every other hue**, when
adjacent, at small size, and under the common color vision deficiencies. Also from your
status colors, so a chart series does not read as an error.

**Illustration palette.** Small, deliberately locked, with real tonal range inside each hue
for shadows, midtones, and highlights.

### Why categorical and illustration must not be merged

The obvious move is to reuse one for the other, since both want more hues than the UI
palette. Do not.

- **They optimize for opposing goals.** Categorical needs maximum mutual
  distinguishability. Illustration needs harmony, colors that sit together in one
  composition. Maximally distinguishable hues placed side by side look garish; harmonious
  hues are by definition closer together and harder to tell apart. You cannot maximize both
  in one set.
- **Breadth versus depth.** Categorical is wide and shallow: many hues, one or two steps
  each. Illustration is narrow and deep: few hues, many tones each. Opposite shapes.
- **Dark mode behaves differently.** Categorical colors sit on a background that changes,
  so they need dark-mode variants. Illustrations are decorative and should be **frozen**,
  not recoloured per theme. See `visual-language.md`, where illustrations are decorative
  and carry `aria-hidden`.

**What all three share:** the same primitive foundation and the same hue families, so
nothing clashes across surfaces. Shared root, divergent branches. An illustration drawn
from the same hue family as the brand ramp will sit correctly next to product UI without
anyone tuning it.

### What a real generated UI palette looks like

Concrete, because abstract rules lose to concrete tables:

```
brand        12 steps   full ramp
neutral      12 steps   full ramp, used more than everything else combined
success      12 steps   full ramp
warning      12 steps   full ramp
danger       12 steps   full ramp
info         12 steps   full ramp
------------------------------------------------------------------
             72 primitives
```

Around **72** for a typical single-brand product. Drop to three status hues and it is 60;
add a defined accent and it is 84. **60-85 colour primitives is the honest range.** This is a colour-only number. Non-colour
primitives (spacing, radius, border width, elevation, z-index, breakpoints, opacity,
motion) are budgeted separately in `tokens.md` and add roughly 40-60 more. Compare a twelve-hue
matrix at 144, which is roughly double for hues nobody named a job for.

The saving does not come from shortening ramps. It comes from not generating the six to
eight hues that had no purpose.

### The generation procedure

1. **Write the semantic token list first.** Surface, content, border, action, feedback
   families.
2. **Derive the hue list from it.** Which distinct hues do those semantics actually
   require? That list is usually shorter than instinct suggests.
3. **For each hue on the list, generate the full 12-step ramp.** Do not truncate.
4. **If a hue cannot be justified from the semantic list, do not generate it.** Not as a
   partial ramp either. It either has a job or it does not exist.
5. **Decide separately whether you need a categorical palette or an illustration palette.**
   Those are their own decisions with their own constraints, not an extension of this one.
6. **Count the hues and state the number out loud before building.** More than about seven
   UI hues for a single-brand product means something is being generated speculatively.

### The self-check

The question is about hues, not steps:

> **What job does this hue do, and which semantic tokens depend on it?**

Any hue with no answer should not exist, at any depth. Within a hue that does have a job,
the honest answer for individual steps is usually "all twelve, across states not built
yet," which is exactly why truncating is the wrong economy.

### The honest counter-argument

Multi-brand and white-label systems need more primitive headroom, because a second brand
may land anywhere on the ramp. And generating a ramp is nearly free, since a script
produces it in a second.

Both true, and the resolution is the one this skill applies to token tiers: **generate what
you like in the source, but only expose and document what has a job.** The thing to control
is availability in the design surface, not the existence of values in a build file. Hiding
unused primitives is the same argument as hiding the primitive tier from designers, applied
to hue count.

### Why this matters more with agents

A model given twelve hues will use more of them than a model given five. Constraining the
option set is one of the few reliable ways to constrain output, and it costs nothing. Same
logic as a coarse spacing scale, a small component allowlist, and off-scale values that
fail to compile.

Note the asymmetry in what constraining means here: **fewer hues, full ramps.** A model
that finds no hover step for a color it is already using will invent one.

## What each step is for: the Radix purpose map

**This table is the argument for full depth.** Read down it and notice how few steps are
optional: a component background, its hover, its pressed state, two border weights, a solid
fill, that fill's hover, and two text weights. Those are not twelve arbitrary shades, they
are twelve jobs that recur in real interfaces.

That is why the guidance above says take the whole ramp for any hue that earns one. Use
this table to assign purpose, and use it as a checklist when you are tempted to truncate:
whichever step you drop is the state someone improvises later.

Radix Colors documents an exact, per-step semantic purpose for every step in a 12-step scale. This is worth adopting close to verbatim - it answers "how many steps" and "what is step N for" in one table. Source: [radix-ui.com/colors/docs/palette-composition/understanding-the-scale](https://www.radix-ui.com/colors/docs/palette-composition/understanding-the-scale).

| Step | Use case |
|---|---|
| 1 | App background |
| 2 | Subtle background |
| 3 | UI element background (normal state) |
| 4 | Hovered UI element background |
| 5 | Active / selected UI element background |
| 6 | Subtle borders and separators (non-interactive: sidebars, headers, cards, alerts) |
| 7 | UI element border and focus rings (interactive, subtle) |
| 8 | Hovered UI element border (interactive, stronger) |
| 9 | Solid backgrounds - this is the **purest step**, least mixed with white/black, used for brand surfaces, logos, colored shadows, accent borders |
| 10 | Hovered solid backgrounds (hover state for step-9 surfaces) |
| 11 | Low-contrast text |
| 12 | High-contrast text |

Steps 1-2 are backgrounds, 3-5 are component backgrounds by state (rest/hover/pressed), 6-8 are borders by strength/interactivity, 9-10 are solid/brand fills, 11-12 are text. **This step-to-purpose mapping is more valuable than the step count itself** - copy the mapping even if you settle on a different number of steps.

## How many steps, and why systems disagree

| System | Steps | Rationale |
|---|---|---|
| Radix Colors | 12 | Each step has one documented semantic purpose (table above) |
| Tailwind CSS | 11 (50-950) | General-purpose scale, not purpose-mapped per step |
| Material 3 (HCT tonal palette) | 13 tones (0,10,20...100 plus a few) | Tones are perceptually even steps in Hue-Chroma-Tone space, not usage-mapped like Radix |
| IBM Carbon | 10 | Simplicity over exhaustive state coverage |

There's no single correct number; what matters is whether the scale gives you enough resolution to express rest/hover/pressed/selected/disabled *and* text-on-surface contrast pairs without doubling back to ad hoc values. 10-12 steps is the practical range used across serious systems.

## Color spaces: why HSL lies to you

- **HSL is not perceptually uniform.** Two colors with the same "L" (lightness) value in HSL can look very different in actual perceived brightness depending on hue - e.g. HSL yellow at L=50% looks far lighter than HSL blue at L=50%. This makes HSL a bad basis for generating an evenly-stepped ramp.
- **OKLCH / OKLAB** fix this: lightness in OKLCH tracks actual perceived lightness consistently across all hues, so a ramp built by stepping "L" alone looks evenly graduated regardless of hue. Tailwind CSS v4 rebuilt its entire default palette in OKLCH for this reason.
- **Material's HCT (Hue, Chroma, Tone)** is Google's own perceptually-accurate space, purpose-built for generating Material's tonal palettes; it's a design-tool-side variant of the same underlying idea as OKLCH/CAM16.
- **CIELAB/LCH** are the earlier (CIE) generation of the same perceptual-uniformity idea; OKLAB is a refinement with fewer known hue-linearity issues.
- Practical rule: **never generate a color ramp by eye in HSL**. Use a tool that operates in a perceptual space (see Tools below), or if hand-authoring in code, compute in OKLCH and only convert to sRGB/hex at the very last step for browser compatibility.

## Tools actually used in production

- **Radix Colors** - pre-built, accessibility-considered, purpose-mapped 12-step scales, light+dark pairs included. Best default if you don't need a fully custom brand hue system.
- **Adobe Leonardo** (leonardocolor.io) - contrast-driven palette generation: you specify a target contrast ratio against a background, and it computes the color that hits it, across an entire ramp. This inverts the usual workflow (pick colors, then check contrast) into (pick contrast targets, generate colors) - the more rigorous approach for accessibility-first palettes.
- **Material Theme Builder / HCT** - generates a full tonal palette + light/dark schemes from a single seed color, used by Material 3's dynamic color system.
- **Lyft's ColorBox** - an early, influential tool for generating full ramps (including dark-mode counterparts) from a handful of input hues; still cited as a case-study pattern for building perceptually consistent light+dark pairs together.
- **chroma.js / culori** - code-level libraries for perceptual color math (OKLCH conversion, interpolation, contrast calculation) when building a custom in-house token generator.

## Accessibility: WCAG 2, its flaws, and what's coming

- **WCAG 2.x contrast ratios**: 4.5:1 for normal text, 3:1 for large text (>=18pt or >=14pt bold) and for non-text UI components/graphical objects (WCAG 2.2 SC 1.4.11). These are the current legally-referenced, tool-checkable standard - build to at least this floor.
- **Known mathematical flaw in WCAG 2's contrast formula**: it's based on a simple luminance ratio that doesn't match actual human perception well at the extremes - it under-penalizes some light-on-light and dark-on-dark pairs and over-penalizes some others, especially with saturated colors. This is well documented in accessibility circles and is the entire motivation for APCA.
- **APCA (Accessible Perceptual Contrast Algorithm)**, the contrast model proposed for WCAG 3, models perceived contrast more accurately (accounts for font size/weight, and light-text-on-dark vs dark-text-on-light asymmetry). As of this research, **WCAG 3 / APCA is not yet a ratified, legally normative standard** - treat it as directionally correct and worth testing against, but do not replace WCAG 2.x compliance with APCA-only compliance for anything with legal/compliance exposure (a real constraint for B2B/fintech/enterprise). Reference: [git.apcacontrast.com](https://git.apcacontrast.com).
- **Focus indicators** (WCAG 2.2 SC 2.4.11/2.4.13): focus indication must not be obscured and must meet a minimum contrast/size against both the unfocused and focused states. A design system's `border.focus` token and focus-ring component behavior should be centrally guaranteed - don't leave this to individual product teams.
- **Color blindness / never color-alone.** Roughly 1 in 12 men and 1 in 200 women have some form of color vision deficiency (deuteranopia most common). Never encode meaning in color alone - status, required-field, error states, and data-viz categories must carry a second channel (icon, shape, text label, pattern).
- **Dark mode contrast** needs its own pass, not an assumption that "light mode passed, so dark mode will too" - see `tokens.md` for the 15.8:1 base-surface rule that guarantees downstream elevation steps stay compliant.

## Palette composition for a real product

A production palette needs, at minimum:

1. **Primary/brand** - the one hue used to signal "this is interactive/on-brand." Often the accessibility bottleneck: bright yellows, limes, and light oranges frequently fail 4.5:1 as text/icon color on white. **Standard fix**: don't force the literal brand hue to serve as your accessible "primary action" color. Either (a) use a darker/higher-chroma step from the same hue family for text/icon/interactive-foreground use and reserve the literal brand hue for large fills/logos/marketing surfaces, or (b) treat brand color as decorative-only and pick a separate, accessible "action" color for interactive elements. A useful discipline either way: reserve the brand hue for elements the user can actually act on, never for decorative fills, so "brand-colored" and "interactive" stay the same signal.
2. **Neutrals** - the largest and hardest-won ramp in any system, because it's used far more than brand color (backgrounds, borders, body text, disabled states). Decide warm/cool/true-neutral early: pure grey (no hue) is safest for multi-brand/white-label systems; a neutral tinted slightly toward the brand hue (e.g. a warm grey if brand is orange) reads as more "designed" but complicates white-labeling later.
3. **Semantic/status** - success, warning, danger, info at minimum. Each needs a **full ramp**, not a triplet. Status colors carry surfaces, hovers, borders, solid fills, fill hovers, and text, and a destructive Button alone exercises most of the scale. See the corrected guidance at the top of this file.
4. **Categorical and illustration palettes** - separate from UI color, and separate from each other (see the three-palette model above). Categorical covers chart series, task labels, and user avatars, with its own constraints: categorical palettes need hues distinguishable from each other *and* from your semantic status colors (so a chart series in red doesn't read as "error"), sequential/diverging palettes need perceptually even steps (again, generate in OKLCH/LCH, not HSL). This is frequently skipped in from-scratch builds and then improvised per-chart, causing inconsistency across dashboards. Razorpay's Blade explicitly ships a dedicated "Chart Color Themes" doc separate from its UI tokens - a concrete signal that mature systems treat this as its own workstream.

## B2B / B2C / internal tools implications

- **B2B / enterprise**: favor restraint and density. More semantic status colors are needed (multi-state workflows, approval pipelines, risk levels) but less decorative saturation. Data-table and form-heavy UI benefits from a slightly desaturated overall palette so status colors (which need to pop) retain contrast headroom.
- **B2C**: more room for brand expression in large surfaces (hero sections, marketing pages, imagery), but the same accessible-action-color discipline still applies to anything actually clickable.
- **Internal tools**: bias toward the plainest, highest-contrast, most legible neutral-dominant palette. Internal tools are used for hours at a stretch by a captive, non-choosing audience - legibility and low eye strain beat brand personality every time.

## Contested / where experts disagree

- **12-step (Radix, purpose-mapped) vs 10-11 step (Tailwind/Carbon, general-purpose) ramps.** Not really a disagreement about correctness, more about whether you want a scale with baked-in usage guidance (Radix) or a blanker scale you map yourself (Tailwind). For a from-scratch 10k+ user build, the Radix step-purpose table is worth adopting regardless of exact step count chosen.
- **WCAG 2 vs APCA.** WCAG 2 is legally referenced today; APCA is more perceptually accurate but not yet normative. Don't drop WCAG 2 compliance in favor of APCA-only for anything with compliance exposure.
- **Whether to generate full ramps for status hues.** Settled here in favor of full depth, on the grounds that a truncated ramp gets improvised past. The counter-position is that a partial ramp is a forcing function that surfaces unplanned states early. Reasonable, but it fails at the wrong moment: under deadline, when someone needs a hover and invents one.
- **Whether the categorical palette should be part of the design system at all.** Some teams treat it as a data-viz concern owned by the charts library. Defensible when charts are the only consumer, less so once user avatars and task labels need the same hues.
- **True neutral grey vs brand-tinted neutral.** Multi-brand/white-label systems should default to true neutral (portable across brands); single-brand consumer products often deliberately tint neutrals warm or cool to reinforce brand feel.

## Common failure modes

1. Picking a primary/brand hue for its marketing appeal without checking it can serve as an accessible interactive-foreground color (bright yellow/lime/light-orange brands hit this constantly).
2. **Generating and exposing hues nobody named a job for.** Every unused ramp is an invitation, and the surplus becomes a license to improvise. Derive the hue list from the semantic tokens, not from what the generator can produce.
3. **Truncating a ramp to save tokens.** The opposite failure, and the more expensive one. A missing hover or border step gets invented inline, which is drift with extra steps. Fewer hues, full ramps.
4. **Merging the categorical and illustration palettes** because both wanted more hues. They optimize for opposing goals: distinguishability versus harmony.
5. **Recolouring illustrations per theme.** Illustrations are decorative and should be frozen across modes; only categorical and UI colors need dark variants.
6. Building the UI palette and the categorical palette as the same thing - chart colors then collide with status colors, and a red series reads as an error state.
7. Generating a ramp in HSL by eye, producing uneven perceptual steps that look "off" without anyone being able to say exactly why.
8. Skipping a genuinely separate accessibility pass for dark mode, assuming light-mode contrast compliance transfers.
9. Encoding a status or requirement purely in color with no secondary indicator (icon/text/pattern), failing color-blind users.
