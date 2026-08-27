# Token architecture, dark mode, and multi-theme reference

## The three tiers, and what each one is actually for

Every mature system converges on the same three-tier model, even though the names differ.

| Tier | Material 3 | Salesforce/EightShapes | What it is |
|---|---|---|---|
| 1 | Reference tokens (`md.ref.*`) | Global / primitive tokens | Raw values. `blue.90`, `space.400`, `#E8DEF8`. No meaning, just a scale. |
| 2 | System tokens (`md.sys.*`) | Alias / semantic tokens | Meaning. `color.background.primary`, `surface.raised`. Points to tier 1. Swaps per theme/mode. |
| 3 | Component tokens (`md.comp.*`) | Component-specific tokens | `button.container.color`. Points to tier 2 (ideally). Only exists so one component can change without touching the semantic layer. |

Source: [m3.material.io/foundations/design-tokens](https://m3.material.io/foundations/design-tokens). Material's own diagram: a button's container color is stored in a component token, which points to a system token, which points to a reference token, which resolves to a hex value. Tokens can be tagged with a **context** (dark theme, dense layout, RTL) and the same system token resolves differently per context.

**Rule of thumb on tier 3:** most teams don't need it until they have >1 platform or a component whose visual identity diverges from the semantic layer often enough to cause churn. Adding it too early is the single most common over-engineering mistake (see Contested below). Start with 2 tiers (primitive -> semantic). Promote a semantic token to a component token only when two different components need to diverge from the same semantic value.

## How many tokens should you actually have

There is real disagreement here, captured directly from practitioners on r/DesignSystems ([reddit.com/r/DesignSystems/comments/1ihf57j](https://www.reddit.com/r/DesignSystems/comments/1ihf57j/am_i_missing_something_or_are_we_overengineering/)):

- One practitioner: "You don't need more than 100-200 tokens in a system, and even that might be overkill... I worked with one client that had ~1500 tokens for the text field alone. That's way way way too many."
- Counter: "laughs in white-label multi-device" - multi-brand and multi-platform genuinely need more.
- Consensus that emerged: **the complexity is fine if it's hidden.** If every component token is linked to a semantic token, and every semantic token is linked to a primitive, "there's no room for error, you use the right tools, it's simple overrides nothing more." The failure mode isn't the tier count, it's exposing all tiers to designers at once in Figma, which "leads to errors and slows down workflows."
- Reference points from real shipped systems: Instacart's live site exposes 64 CSS custom properties, already semantic. Blinkit exposes 222-282, mostly raw primitives with an internal ramp scheme and no semantic layer. That contrast is the evidence: skipping the semantic tier is what makes token counts balloon, because every consumer then references primitives directly and nothing can be themed cleanly.

**Practical target for a single-brand, single-platform system at 10k+ users:** roughly 40-80 primitives, 60-120 semantic tokens, component tokens added only as needed. If you're building multi-brand or multi-platform from day one, budget 2-4x that, but keep it in the primitive layer (more brand palettes), not the semantic layer (semantic names should stay stable across brands).

## Dark mode: do not invert, build a second value set

The single most-repeated failure mode across every source: treating dark mode as "invert the light palette." It does not work because:

1. **Pure black is wrong.** Material's own guidance: use `#121212` (dark grey), not `#000000`. Reasons, verbatim from Material: dark grey surfaces "can express a wider range of color, elevation, and depth, because it's easier to see shadows on grey (instead of black)," and light text on dark grey has lower contrast than on pure black, reducing eye strain. [m2.material.io/design/color/dark-theme.html](https://m2.material.io/design/color/dark-theme.html)
   - Exception: OLED battery-constrained contexts (wearables) can justify true black, but accept the tradeoff (motion blur when pixels switch, per multiple sources including [atmos.style/blog/dark-mode-ui-best-practices](https://atmos.style/blog/dark-mode-ui-best-practices)).
2. **Saturated colors vibrate on dark backgrounds.** Desaturate brand/accent colors for dark mode - roughly 20 points of saturation lower than the light-mode equivalent is the commonly cited rule of thumb ([atmos.style](https://atmos.style/blog/dark-mode-ui-best-practices)). Material's own dark theme properties state colors must be "desaturated so they pass WCAG AA (4.5:1)... at all elevation levels."
3. **Elevation flips from shadow to lightness.** In light mode, higher elevation = bigger shadow. In dark mode, shadows barely register against a dark background, so Material's answer is: **higher elevation = lighter surface**, via a semi-transparent white overlay whose opacity increases with elevation:

   | Elevation | White overlay opacity |
   |---|---|
   | 0dp | 0% |
   | 1dp | 5% |
   | 2dp | 7% |
   | 3dp | 8% |
   | 4dp | 9% |
   | 6dp | 11% |
   | 8dp | 12% |
   | 12dp | 14% |
   | 16dp | 15% |
   | 24dp | 16% |

   **Important version note:** the overlay table above is Material *2*. Material *3* replaced the semi-transparent-overlay mechanic with **tonal elevation**: a set of discrete named surface tones (`surface`, `surface-container-lowest`, `surface-container-low`, `surface-container`, `surface-container-high`, `surface-container-highest`) that you pick directly rather than computing an overlay. If you inherited an `md-sys-color-surface-tint` token from an older Material Theme Builder export, that is a legacy artifact of the M2 overlay model, not current M3 practice. The M2 opacity table is still the clearest published explanation of *why* lightness encodes elevation on dark, which is why it is reproduced here; implement it as discrete surface tokens rather than runtime overlays.

   Source: [m2.material.io/design/color/dark-theme.html](https://m2.material.io/design/color/dark-theme.html). Overlays are **not** applied to surfaces already using a primary/secondary brand color - only to neutral surface containers. Don't use light glows instead of shadows; they don't read as elevation.
4. **On-brand dark backgrounds.** To make a dark surface feel branded instead of generic slate, blend a low-opacity version of your primary color into the dark base: `#121212` + 8% primary = a branded dark base, per Material. A practitioner case study reproduced this exactly for a fintech ("Tenet UI"): base dark background `#09111A` chosen so pure white text still hits 15.8:1, giving headroom for every lighter elevation step to still pass 4.5:1 ([fourzerothree.in/p/scalable-accessible-dark-mode](https://www.fourzerothree.in/p/scalable-accessible-dark-mode)).
5. **The 15.8:1 rule.** Material's justification for how dark a base surface must be: white text needs at least 15.8:1 contrast against the *deepest* background so that at the *lightest* (most elevated) surface, contrast still clears WCAG AA's 4.5:1 floor. Work backward from your highest elevation surface, not forward from your base.
6. **"On" colors carry hierarchy via opacity, not new hues.** Material's dark theme text hierarchy: high-emphasis text at 87% white opacity, medium/hint at 60%, disabled at 38%. High-priority text is deliberately not pure `#FFFFFF` because a fully bright color visually "vibrates" against a dark background ([codelabs.developers.google.com/codelabs/design-material-darktheme](https://codelabs.developers.google.com/codelabs/design-material-darktheme)).
7. **Shadows need their own dark-mode values.** A shadow authored for a white background is often invisible or wrong on a dark one. If your token pipeline handles color modes but not shadow tokens per mode, you will ship dark cards with no visible separation. ([zeroheight.com/learn - Figma modes guide](https://zeroheight.com/learn/figma-variable-modes-in-depth-theming-dark-mode-and-brand-switching/))

**Practical build sequence for dark mode:** author the semantic layer first (surface/content/border/action families), design light mode fully, then author a **second value set for the same semantic names** - not a mechanical inversion. Test the darkest and lightest elevation extremes for contrast before filling in the middle steps. Treat dark mode as equally first-class from day one if you can; retrofitting it later means re-touching every component's hover/pressed/disabled states, which is where most of the real bugs hide. A specific trap to expect: lightening hover and pressed states is the intuitive dark-mode move and it routinely destroys contrast against light labels, so a dark theme can introduce accessibility failures the light theme never had.

## Theme axes beyond light/dark

Real systems stack more than one axis: mode (light/dark), density (comfortable/compact), brand (multi-tenant/white-label), platform (web/iOS/Android/RN), and occasionally a high-contrast/accessibility mode, which is genuinely distinct from dark mode and should not be collapsed into it.

**Keep theme and mode as separate, composable dimensions.** Nathan Curtis draws this distinction explicitly: a theme (a brand or product line) is orthogonal to a mode, and a single theme can have its own light *and* dark variant. His naming shape for this is `$system-theme-category-concept-property-on-mode`, e.g. `$aads-ocean-color-forms-text-metadata`. Collapsing brand and mode into one axis is what makes token counts explode combinatorially.
 The way to avoid combinatorial explosion, per zeroheight's token guide and confirmed by Blade's own architecture: **only the semantic layer branches per axis; the primitive layer is shared.** A brand-mode swap should only ever remap which primitives the semantic aliases point to, never invent new semantic names per brand. Razorpay's Blade did exactly this for its Payment vs Banking brand themes and light/dark, then collapsed onto **Figma Variables specifically to kill combinatorial mode explosion** - before variables, multiple modes (light/dark) times multiple themes (payment/banking) times every component caused "massive memory consumption" and slow files; after variables, "we have been able to streamline Blade into a single theme which massively boosts designer productivity." ([figma.com/customers/razorpay](https://www.figma.com/customers/razorpay-boosting-design-system-adoption-and-collaboration/))

## Tooling and pipeline

- **W3C Design Tokens Community Group (DTCG) format** reached its first stable version, **2025.10**, in October 2025 ([designtokens.org](https://www.designtokens.org/), [w3.org/community/design-tokens](https://www.w3.org/community/design-tokens/2025/10/28/design-tokens-specification-reaches-first-stable-version/)). Tokens are JSON with `$type`, `$value`, `$description` keys; a token can alias another with `{token.path}` syntax. This is now the interchange format Figma variables, Style Dictionary, and Tokens Studio all converge on - author in this format if you want tool portability.
- **Style Dictionary v4+** has first-class DTCG support and is the dominant build pipeline: JSON tokens in -> platform-specific outputs (CSS custom properties, iOS, Android, SCSS) out. [styledictionary.com](https://styledictionary.com/), [github.com/style-dictionary/style-dictionary](https://github.com/style-dictionary/style-dictionary). Full DTCG 2025.10 support is still landing in v5 as of this writing - check before assuming 100% coverage.
- **Figma Variables** are the design-tool-side equivalent of tokens, organized into **collections** with **modes** (Light/Dark, Brand A/B). **Mode limits are per plan and were raised on 28 October 2025:**

  | Plan | Variable modes per collection |
  |---|---|
  | Starter | Not available |
  | Professional | Up to 10 |
  | Organization | Up to 20 |
  | Enterprise | Unlimited (via extended collections) |

  Source: [help.figma.com - Figma plans and features](https://help.figma.com/hc/en-us/articles/360040328273-Figma-plans-and-features). **Verify these before architecting around them; Figma has changed them at least once and older articles still quote the previous 4-per-collection / 40-on-Enterprise limits.** If you genuinely exceed your plan's mode count, the escapes are group-based organization within one mode, separate semantic collections per brand, or Tokens Studio.

  Note the design implication: on Professional, 10 modes means you can carry roughly 5 brands across light and dark before you run out. That is a real architectural ceiling for a white-label product, and a reason to keep brand and mode as separate collections rather than one multiplied matrix.
- **Common mistake in Figma:** applying a mode override on a deeply nested component instead of at the page/frame level - the mode then doesn't cascade to siblings. Apply mode switches at the highest relevant container.
- **Single source of truth:** the practical answer most systems converge on is **code is the source of truth for values, Figma variables mirror it** - not the reverse - because code is what ships and what CI can lint. Figma-first works only if you have a reliable export pipeline (native DTCG export, or Tokens Studio) feeding Style Dictionary; otherwise design and code silently drift.

## Non-color token types

- **Spacing.** Two real camps: 4pt grid (`4, 8, 12, 16, 20, 24...`) for fine component-level control, and 8pt grid (`8, 16, 24, 32...`) for page/layout rhythm. Most production systems use a hybrid: a 4px base unit but only expose the even multiples (effectively an 8pt rhythm) as the default scale, with 4px reserved for icon-to-label gaps and similar micro-adjustments. Keep the scale deliberately coarse (8-10 steps) - a token scale with 20+ spacing steps is a sign designers are eyeballing values instead of composing from the scale.
- **Typography.** Use **composite tokens**: one token bundles font family, size, weight, line height, letter spacing, and case, rather than four separate atomic tokens a designer must remember to combine consistently. Material 3 does this: each of its ~30 type-scale styles is "a single token" combining all properties ([m3.material.io/styles/typography/type-scale-tokens](https://m3.material.io/styles/typography/type-scale-tokens)). This is also DTCG's own recommendation - composite tokens for "closely related style properties that are always applied together" ([designtokens.org/tr/drafts/format](https://www.designtokens.org/tr/drafts/format/)).
- **Motion.** Atlassian's model is a clean reference: motion tokens like `motion.popup.enter` bundle **duration + easing curve + property** into one semantic decision, named by *intent* not by raw values, so designers/engineers "choose motion by intent rather than raw values" ([atlassian.design/foundations/motion](https://atlassian.design/foundations/motion)). Material 3 buckets duration into short (50-200ms, for hover/press feedback), medium (250-400ms, for on-screen transitions), long (450-600ms, for large expressive transitions), extra-long (700-1000ms, rare, ambient) ([m3.material.io/styles/motion/easing-and-duration](https://m3.material.io/styles/motion/easing-and-duration/tokens-specs)). Always pair a motion system with a reduced-motion fallback (Atlassian explicitly calls this out as a principle, not an afterthought).
- **Radius, border width, z-index, breakpoints, opacity** should all be scales, same discipline as spacing: a small number of named steps, no arbitrary values allowed to compile (enforce with a Tailwind arbitrary-value lint, or better, make off-scale utilities fail to compile).

## Governance: changing tokens safely at scale

- Treat token changes like an API: a renamed or removed *semantic* token is a breaking change and needs a deprecation window, not a silent rename. Keep the old token as an alias to the new one for at least one release, and instrument which teams still reference it (prop and token usage instrumentation via `react-scanner` or equivalent is the concrete mechanism, see `evaluation.md`).
- Primitive-layer changes (adjusting a hex value inside a ramp) are lower risk if every consumer only ever touches the semantic layer - this is the entire argument for the semantic tier existing.
- Version tokens alongside the component library, not separately; a token package one major version behind the component package is a very common real-world source of visual drift.

## Contested / where experts disagree

- **2-tier vs 3-tier tokens.** Material, Salesforce, Adobe all publish 3-tier models as the ideal. Working practitioners on Reddit push back hard that 3-4 tiers exposed directly to designers in Figma is what actually causes error-proneness, not a inherent property of having tiers - the fix is tooling/UI discipline (hide primitives from the component-authoring surface), not fewer tiers. Resolve for your team: adopt 3 tiers, but only ever expose semantic + component tokens in files designers touch day to day.
- **True black vs dark grey for OLED.** Material recommends dark grey (#121212) as the default and explicitly permits true black only for battery-constrained/OLED niche cases. Apple's own dark mode instead defaults to pure black (`#000000`), a real platform-level disagreement - if you're building web/cross-platform rather than native iOS, default to Material's grey-based reasoning; it's the more broadly defensible choice for text-heavy UI.
- **Figma-first vs code-first as source of truth.** No fully resolved consensus; the safest default for a 10k+ user product company is code-as-truth with a synced Figma mirror, because code is what CI can enforce and what ships.

## Common failure modes (cross-checked across sources)

1. Inverting a light palette mechanically to "make" dark mode - breaks contrast and elevation every time.
2. Pure black/pure white as base surfaces, causing vibration and glare.
3. Forgetting shadow tokens need dark-mode-specific values (a light-mode shadow is often invisible on dark).
4. Exposing every token tier to every designer, causing decision paralysis (the "1500 tokens for one text field" story).
5. Skipping the semantic tier entirely and consuming primitives directly in components - this is what makes token counts balloon into the hundreds with no way to theme cleanly (Blinkit's 222-282 raw tokens vs Instacart's 64 semantic ones).
6. Applying a Figma mode override at the wrong (too-nested) layer so it doesn't cascade.
7. Treating token architecture as a one-time decision instead of something instrumented and monitored (see Blade's usage-tracking approach).
