# Naming conventions reference: tokens, components, variants

## Nathan Curtis / EightShapes token naming model (the canonical framework)

Source: [medium.com/eightshapes-llc/naming-tokens-in-design-systems](https://medium.com/eightshapes-llc/naming-tokens-in-design-systems-9e86c7444676). Curtis has architected tokens at Discovery Education, Morningstar, REI, NetApp, and ~10 other systems; this article is the field's most-cited reference on the topic.

Even a simple token exhibits **levels**:

- `$esds-color-neutral-42` -> **namespace** (`esds`) + **category** (`color`) + **variant** (`neutral`) + **scale** (`42`) -> resolves to `#6B6B6B`.
- `$esds-space-1-x` -> namespace + category + scale -> `16px`.

More complex tokens add **concept** and **property** levels between category and variant, e.g. distinguishing "what kind of thing" (concept: background, border, text) from "which visual property" (property: color, width) from "which variant" (brand, danger, neutral) from "which state" (hover, disabled) from "which step on the scale."

**General ordering principle across every system studied: general to specific, left to right.** Namespace first (which system), then category (color/space/type/motion), then narrowing qualifiers, then the most specific/variable part (scale step or state) last.

## Side-by-side: the same concept, named differently across real systems

| Concept | Material 3 | Atlassian | Shopify Polaris | Salesforce | Style Dictionary convention |
|---|---|---|---|---|---|
| Namespace/prefix | `md.` | (none, package-scoped) | `--p-` | (Lightning Design System tokens, no fixed prefix shown) | project-defined, e.g. `esds-` |
| Tier marker | `sys` / `ref` / `comp` embedded in path | flat, no tier marker in the name itself | flat | tiered internally, not always in the literal name | `category-type-item` (CTI) structure |
| Example: color on a colored surface | `md.sys.color.on-primary-container` | `color.text.oninverse` / similar `on*` pattern | `--p-color-text-on-bg-fill-brand` (post-tokenized) | uses `.on*` convention documented at Clarity 2019 (Ferrua & Rewis) | `color-text-on-*` |
| Example: subtle danger fill | (uses `error-container`) | `color.background.accent.blue.subtlest` (real pattern: hue + subtlety qualifier) | `color-bg-fill-critical-subdued` (pattern) | n/a | `color-background-danger-subtle` |
| Separator | dot (`.`) | dot (`.`) in design-token form, becomes hyphen in CSS custom property | hyphen (`-`) | varies | hyphen for CSS vars, dot for JSON tree |
| Motion example | `md.sys.motion.duration.short1`, `md.sys.motion.easing.emphasized` | `motion.popup.enter` (bundles duration+easing+property as one semantic name) | n/a documented | n/a | n/a |

Key takeaways from the comparison:
- **Dot-separated hierarchical names in the token *definition* (JSON/DTCG), hyphen or camelCase in the shipped *output*** (CSS custom properties use hyphens; JS/JSON objects use camelCase) is the standard pattern, not a contradiction - Shopify's own docs state this explicitly: "In JavaScript, design token names are formatted in lower camelCase... In JSON, kebab-case." ([github.com/Shopify/polaris-tokens](https://github.com/shopify/polaris-tokens))
- **The `on-*` / `content-on-*` prefix pattern** for "foreground color guaranteed legible on top of this specific background" is close to universal (Material's `on-primary`, `on-surface`; Atlassian and Salesforce equivalents). Adopt this pattern directly - it solves the extremely common bug of a text/icon color token being reused on a background it wasn't designed to sit on.
- **Atlassian's motion naming is a better model than raw duration tokens**: naming by *where it's used* (`motion.popup.enter`) rather than by *what value it is* (`duration-200-ease-out`) means the design decision, not the literal number, is what a consumer references - you can retune the actual ms/curve without anyone needing to touch component code.

## Concrete naming rules, distilled

1. **Order general to specific.** `color.background.danger.subtle`, never `danger.subtle.background.color`.
2. **Separator conventions**: dot in the token definition tree (supports nesting/aliasing cleanly), hyphen in CSS custom properties (`--color-background-danger-subtle`), camelCase in JS.
3. **Value-in-name vs abstracted name.** `blue-500` (descriptive) is correct **only at the primitive tier**. `surface-danger` (semantic) is correct at tier 2+. Never let a semantic-tier token literally encode a color name (`text-blue` is a primitive-tier name masquerading as semantic - if the brand color changes, every consumer of `text-blue` breaks even though nothing about "this text should look linked" changed).
4. **State vocabulary.** The vocabulary that's converged across systems: `rest` (or the state name is simply omitted for default) -> `hover` -> `pressed` (or `active`) -> `focus` -> `disabled` -> `selected`. Use exactly these words; avoid synonyms like `active` vs `pressed` meaning the same thing in different components - pick one and apply it everywhere.
5. **Avoid words that date badly.** Never name a token or component `new`, `v2`, `alt`, `updated` - these are meaningless in 18 months and force an awkward rename later. Encode versioning in your release process, not the name.
6. **Primary/secondary/tertiary ambiguity.** These words get overloaded to mean both "visual hierarchy of emphasis" (button prominence) and "brand palette position" (primary color vs secondary color) in the same system, which is confusing. Prefer `emphasis` or `hierarchy` language (`primary` / `secondary` / `tertiary` action buttons) for component-level hierarchy, and keep those words entirely separate from palette-naming (`brand`, `accent`, `neutral` for palette roles).

## Component naming: the classic collisions, and the tie-breaker

These names are genuinely contested across the industry, with different systems making different (defensible) choices:

| Ambiguous pair/group | What the WAI-ARIA Authoring Practices Guide (APG) actually calls it |
|---|---|
| Modal vs Dialog vs Sheet | **Dialog (Modal)** - APG's own pattern name is literally "Dialog (Modal)". "Sheet" is a platform-specific (usually mobile) visual variant of the same underlying pattern, not a distinct ARIA role. |
| Dropdown vs Select vs Menu vs Combobox | APG has three **distinct, non-interchangeable** patterns: **Listbox** (a list of options, no free text entry), **Menu and Menubar** (a list of actions/commands, not selectable data), **Combobox** (an input with an associated popup, supports free text + filtering). "Dropdown" is not an ARIA pattern name at all - it's a visual description that gets misapplied to all three. Pick the APG pattern that matches the actual interaction (selecting a value = Listbox/Select; triggering an action = Menu; typing to filter = Combobox), then name your component after that, not after "Dropdown." |
| Toast vs Snackbar vs Banner vs Alert | APG has **Alert** (brief, important, non-interrupting message) and **Alert and Message Dialogs** (interrupting, modal, requires a response). "Toast," "Snackbar," and "Banner" are all product-vocabulary variants of the *non-modal* Alert pattern, differentiated by placement/persistence/dismissal behavior, not by ARIA role. Recommendation: name the component after its actual behavior (does it self-dismiss? is it modal? is it page-level and persistent, or transient and floating?), and document which one is which rather than inventing new ARIA-adjacent vocabulary. |
| Chip vs Tag vs Pill vs Badge | Not an APG pattern at all - purely a design-system-vocabulary decision. The distinguishing test that resolves it cleanly: **if the user can click it and it has a selected/toggled state, it's a Chip (interactive filter/category); if it's passive metadata the user cannot act on, it's a Badge.** "Pill" and "Tag" are usually just shape/visual synonyms layered on top of one of those two behavioral categories - avoid having both a Chip and a Tag and a Pill as three separate components; that's the classic naming-smell of accidental duplication. |
| Tooltip vs Popover | APG has both as **distinct** patterns: **Tooltip** triggers on focus/hover only, is non-interactive, purely informational. **Popover isn't in the current APG pattern list directly** but is universally understood as a richer, often-interactive, click-triggered overlay. If content inside needs to be clicked/focused, it cannot be a Tooltip (a11y-wise) - it must be a Popover-pattern component. |
| Drawer vs Sidebar vs Sheet | Not an APG pattern; distinguish by permanence (Sidebar = persistent layout region) vs transience (Drawer/Sheet = temporarily overlays content, dismissible). |

**General rule: align component naming to ARIA role/pattern vocabulary wherever a matching APG pattern exists, and only invent new vocabulary for genuinely non-standard widgets.** This matters more than it sounds like it should, because (a) it keeps your accessibility implementation honest - naming something "Dropdown" invites someone to build a plain `<div>` with a click handler instead of a real Listbox/Combobox, and (b) it's what makes your component names predictable to any new engineer who already knows web platform vocabulary. Full APG pattern list: [w3.org/WAI/ARIA/apg/patterns](https://www.w3.org/WAI/ARIA/apg/patterns).

## Variant and prop naming

- **`variant` vs `kind` vs `appearance` vs `intent`**: all four are used across real systems (Polaris tends toward `tone`, Primer toward `variant`, Spectrum toward `variant`, Material toward baked-in component-specific names rather than one universal prop). There's no single winner; the actual rule that matters is **internal consistency** - pick one word and use it for the same kind of decision (visual style choice) across every component, and reserve a different word (`size`, `state`) for orthogonal axes.
- **Size naming**: t-shirt sizing (`sm`/`md`/`lg`) is more common in component libraries because it's stable under redesign (a numeric scale like `100/200/300` implies precision the design doesn't actually have and forces awkward insertions like `150` later). Numeric scales are more common in raw token scales (spacing, radius) where the numbers *are* the actual pixel values.
- **Boolean props**: `isDisabled` / `isSelected` (explicit `is*` prefix) vs bare `disabled` / `selected`. React/HTML native attribute convention favors the bare form (`disabled` matches the native `<button disabled>` attribute); the `is*` prefix is more common in systems that want boolean props to be visually unambiguous at a glance in JSX. Match whichever your component's underlying platform primitive already uses, to reduce prop-translation friction.
- **Keep Figma variant property names identical to code prop names, verbatim, including casing.** This is the specific practice Razorpay's Blade calls its core philosophy - "what you see in design is what you get in code," with "all component properties on Figma hav[ing] a 1:1 mapping to React props for ease of translation, there is no exception to this philosophy" ([engineering.razorpay.com/cutting-deep-through-blade](https://engineering.razorpay.com/cutting-deep-through-blade-23a72bcc3bcc)). This isn't just a developer-experience nicety anymore - it's the exact property that makes a design system usable by AI codegen/Figma-MCP tooling, since the agent can map a Figma variant string directly to a prop value with zero translation layer.

## Figma layer and component naming

- **Slash-nested naming** (`Button/Primary/Large`) organizes component variants into Figma's component-set browser hierarchically; this predates and is now largely superseded by native Figma **variant properties** (a single component set with `Variant=Primary`, `Size=Large` as separate properties) - prefer real variant properties over slash-naming where your Figma version supports it, since it maps more directly to a props object in code.
- Keep page/frame organization conventions consistent with your token/component taxonomy (e.g. a "Foundations" page mirrors the primitive+semantic token tiers, a "Components" page mirrors the registry) so a new designer's mental model of the Figma file matches a new engineer's mental model of the codebase.

## Contested / where experts disagree

- **`variant` vs `intent` vs `tone` as the prop name for visual style.** No consensus; pick one, be consistent within your system.
- **Slash-naming vs native variant properties in Figma.** Native properties are technically superior once your team is on a Figma version/plan that supports them well; slash-naming persists mostly for legacy/historical reasons and simpler mental model for very small teams.
- **`is*`-prefixed booleans vs bare booleans.** Split by platform convention; no universal winner.

## Naming smells (fix these when you see them)

1. A `primary`/`secondary`/`tertiary` token or prop that's ambiguous between "brand palette position" and "visual emphasis hierarchy" in the same system.
2. Three near-duplicate components (Chip/Tag/Pill) that differ only by visual shape, not by behavior - collapse to one component with a shape variant.
3. A component literally named `Dropdown` - check which APG pattern (Listbox/Menu/Combobox) it should actually be and rename/re-implement.
4. Any token or component name containing `new`, `v2`, `old`, `legacy`, `temp` - these should be release/deprecation metadata, not permanent names.
5. Figma variant property names that don't match the code prop names/casing exactly - a drift source for both humans and AI codegen.
