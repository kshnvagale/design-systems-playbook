# Color palette construction and accessibility reference

## The Radix 12-step model (the single most useful artifact in this space)

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

## Generate only the color you need

Before deciding how many steps, decide how many **hues**. The default instinct is to
generate a full matrix, twelve hues times twelve steps, because the tooling makes it free.
Do not.

**A generated full spectrum is not neutral. Availability is permission.** Designers and
agents both reach for what is in front of them. Ship 12 hues when the product needs 4 and
the other 8 will appear somewhere, uncontrolled, and then have to be policed forever. This
is one of the most reliable failure patterns in a real design team: give people every
color and the colors end up in random places, justified after the fact.

### What a real product actually needs

| Family | How much | Why |
|---|---|---|
| **Brand / primary** | One hue, full ramp | The only hue that usually needs every step, because it carries backgrounds, borders, fills, and text |
| **Neutral** | One ramp, full | Used more than everything else combined: backgrounds, borders, body text, disabled states |
| **Status** (success, warning, danger, info) | 3-4 steps each, not full ramps | Each needs a surface, a border, and a content step. Nothing else. |
| **Accent / secondary** | Only if it has a defined job | If you cannot name what it is for, do not generate it |
| **Data visualization** | Separate palette, only if there are charts | Different constraints entirely, see below |

For a typical product that is roughly **two full ramps plus four partial ones**, not twelve
full ones. Around 40 real color tokens rather than 144 swatches.

### The rule

**Generate a ramp step when a semantic token will point at it. If no semantic token
references a step, do not ship the step.**

The semantic layer is the demand signal. The primitive layer should satisfy demand, not
anticipate it. Work backwards: write the semantic token list first (surface, content,
border, action, feedback families), then generate exactly the primitives those aliases
need.

Adding a step later is trivial. Removing one after it has been used in forty places is
not. The asymmetry should decide this for you.

### The honest counter-argument

Multi-brand and white-label systems genuinely need more primitive headroom, because a
second brand may land anywhere on the ramp. And a full perceptual ramp is nearly free to
*generate*, since a script produces it in a second.

Both are true, and the resolution is the same one this skill applies to token tiers:
**generate the full ramp in the source if you like, but only expose and document the steps
in use.** The thing to control is availability, not the existence of values in a build
file. Hiding unused primitives from the day-to-day design surface is the identical
argument, applied to palette size instead of tier depth.

### Why this matters more with agents

A model given 144 color tokens will use more of them than a model given 40. Constraining
the option set is one of the few reliable ways to constrain output, and it costs nothing.
The same logic runs through this skill: a coarse spacing scale, a small component
allowlist, off-scale values that fail to compile.

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
3. **Semantic/status** - success, warning, danger, info at minimum. Each needs its own mini-ramp (surface/border/content triplet at least) so status colors work as backgrounds, borders, and text, not just a single swatch.
4. **Data visualization palette** - must be a **separate palette** from UI color, with its own constraints: categorical palettes need hues distinguishable from each other *and* from your semantic status colors (so a chart series in red doesn't read as "error"), sequential/diverging palettes need perceptually even steps (again, generate in OKLCH/LCH, not HSL). This is frequently skipped in from-scratch builds and then improvised per-chart, causing inconsistency across dashboards. Razorpay's Blade explicitly ships a dedicated "Chart Color Themes" doc separate from its UI tokens - a concrete signal that mature systems treat this as its own workstream.

## B2B / B2C / internal tools implications

- **B2B / enterprise**: favor restraint and density. More semantic status colors are needed (multi-state workflows, approval pipelines, risk levels) but less decorative saturation. Data-table and form-heavy UI benefits from a slightly desaturated overall palette so status colors (which need to pop) retain contrast headroom.
- **B2C**: more room for brand expression in large surfaces (hero sections, marketing pages, imagery), but the same accessible-action-color discipline still applies to anything actually clickable.
- **Internal tools**: bias toward the plainest, highest-contrast, most legible neutral-dominant palette. Internal tools are used for hours at a stretch by a captive, non-choosing audience - legibility and low eye strain beat brand personality every time.

## Contested / where experts disagree

- **12-step (Radix, purpose-mapped) vs 10-11 step (Tailwind/Carbon, general-purpose) ramps.** Not really a disagreement about correctness, more about whether you want a scale with baked-in usage guidance (Radix) or a blanker scale you map yourself (Tailwind). For a from-scratch 10k+ user build, the Radix step-purpose table is worth adopting regardless of exact step count chosen.
- **WCAG 2 vs APCA.** WCAG 2 is legally referenced today; APCA is more perceptually accurate but not yet normative. Don't drop WCAG 2 compliance in favor of APCA-only for anything with compliance exposure.
- **True neutral grey vs brand-tinted neutral.** Multi-brand/white-label systems should default to true neutral (portable across brands); single-brand consumer products often deliberately tint neutrals warm or cool to reinforce brand feel.

## Common failure modes

1. Picking a primary/brand hue for its marketing appeal without checking it can serve as an accessible interactive-foreground color (bright yellow/lime/light-orange brands hit this constantly).
2. **Generating the full spectrum because the tool made it easy.** Every unused ramp is an invitation, and the surplus becomes a license to improvise. Generate against the semantic token list, not against what the generator can produce.
2. Building the UI palette and the data-viz palette as the same thing - chart colors then accidentally collide with status colors (a "red" data series reading as an error state).
3. Generating a ramp in HSL by eye, producing uneven perceptual steps that look "off" without anyone being able to say exactly why.
4. Skipping a genuinely separate accessibility pass for dark mode, assuming light-mode contrast compliance transfers.
5. Encoding a status or requirement purely in color with no secondary indicator (icon/text/pattern), failing color-blind users.
