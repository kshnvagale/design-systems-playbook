# Mobile web: scaling, layout, and touch

Scope is **mobile web** (browsers on phones and tablets), not native iOS/Android apps.
Everything here assumes the product is responsive across viewports, not that a separate
mobile site exists. Token units are covered in `tokens.md`; this file is layout, input, and
platform behavior.

## The default: mobile-first, content-driven breakpoints

Tailwind is mobile-first by construction: unprefixed utilities apply at all sizes, and
breakpoint prefixes (`sm:`, `md:`) apply *upward* from that size. Author the base case for
the smallest viewport, then add complexity as space allows, rather than designing for
desktop and cramming it down.

**Breakpoints should be set where your content breaks, not where a specific device's
viewport happens to be.** Device sizes change every year; the point where your grid or
your longest heading starts wrapping badly does not. Use device-named breakpoints (`sm`,
`md`, `lg`) as labels, not as a promise that `sm` means "phone."

Tailwind v4 breakpoints are defined as `@theme` variables in `rem`, and are overridden the
same way as any other token:

```css
@theme {
  --breakpoint-sm: 30rem;
}
```

## How scaling works across viewports

**The rule: layout changes across viewports, components mostly do not.** A Button, an
Input, a Card's internal padding, and a touch target stay the same size on a phone and on a
laptop. What changes is how many fit side by side, where the navigation lives, and how much
air sits between page sections. Components adapt to their container through container
queries; tokens stay fixed; only page-level spacing and display type flex.

| What | Scales with viewport? | How |
|---|---|---|
| Display and heading type | **Yes** | Fluid `clamp()` anchored in `rem`, see below |
| Body text | Usually no | Fixed `rem`. Some fluid scales grow body text slightly; see Contested |
| Page and section spacing | **Yes** | Step it at breakpoints, or make only the section-level tokens fluid |
| Grid gaps | Sometimes | Step at a breakpoint; keep the same token family |
| Component internal padding | **No** | Fixed tokens. Change layout around the component, not inside it |
| Touch target hit area | **Never shrinks** | Constant minimum at every viewport |
| Borders, radius, shadows | No | Fixed `px` per `tokens.md` |
| Density | Never denser on touch | Touch needs more room, not less. Desktop with a fine pointer may run denser |

Two consequences for the token layer:

- **Keep spacing tokens fixed.** Do not make the whole spacing scale fluid. If sections need
  more air on large screens, add or step *section-level* tokens, and leave the component
  scale alone. A fluid component scale means a Button is a different size on every device,
  which is exactly the inconsistency a system exists to prevent.
- **If you support density modes, tie them to pointer precision, not viewport width.** A
  tablet in landscape is wide and still touch-driven. `(pointer: fine)` is the honest signal.

## Container queries: the addition that matters most for a component library

A design system ships components that get placed in unknown contexts: a `Card` in a
three-column grid, the same `Card` full-width in a sidebar. A viewport media query cannot
express "respond to the space *this component* actually has." A container query can.

```css
@container (min-width: 24rem) {
  .card { grid-template-columns: 1fr 1fr; }
}
```

Tailwind v4 ships `@container` support natively. Browser support is broad in current
evergreen browsers as of 2026. **Default to container queries for component-level layout
decisions, and reserve viewport media queries for page-level layout** (nav collapse, overall
grid).

## Fluid type: clamp() with rem, never pure vw

A heading that should scale smoothly between a mobile minimum and a desktop maximum can use
`clamp()`:

```css
font-size: clamp(1.75rem, 4vw + 1rem, 3rem);
```

**Anchor the clamp in `rem`, not pure `vw`.** Pure viewport-unit type does not track the
user's browser zoom correctly, which is the same accessibility regression `tokens.md`
already argues against for fixed values. Combining a `rem` floor and ceiling with a `vw`
scaling term keeps zoom working while still scaling with the viewport. Utopia
(utopia.fyi) is the standard method for generating a whole fluid scale this way rather than
hand-tuning each `clamp()`.

**Default to fluid display and heading roles only.** Keep body text a fixed `rem` size. The
default font-size setting already lets users enlarge it, and a viewport-driven body size
adds a second moving part for little readability gain. Some fluid systems (Utopia's
defaults among them) scale body text slightly; that is a legitimate choice, see Contested.

## Viewport units: `100vh` is broken on mobile, use `dvh`

`100vh` on mobile Safari and Chrome includes space the browser chrome (address bar, tab
bar) will occupy, so a full-height element is taller than the visible area and gets cut off
or causes an unwanted scrollbar. This has been a known mobile web bug for years.

Use the dynamic/small/large viewport unit family instead:

| Unit | Behavior | Use for |
|---|---|---|
| `dvh` | Tracks the *actual* visible height as browser chrome shows/hides | Full-height layouts that should never jump |
| `svh` | Height with all browser chrome visible (smallest case) | Modal and bottom-sheet overlays, where you want the guaranteed-safe minimum |
| `lvh` | Height with all browser chrome hidden (largest case) | Rarely needed alone |

```css
.bottom-sheet {
  max-height: 90svh; /* never taller than the guaranteed-visible area */
}
```

Prefer `svh` for anything that must never be obscured by chrome appearing (a bottom sheet,
a fixed CTA), and `dvh` for content that should track the real viewport as it resizes.

## Safe areas: required for anything pinned to an edge

Notches, the Dynamic Island, and the home indicator can overlap fixed headers, bottom
navigation, bottom sheets, and toasts. Handle it with the `env()` safe-area insets and the
viewport meta tag that enables them:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

```css
.bottom-nav {
  padding-bottom: max(1rem, env(safe-area-inset-bottom));
}
```

**Any organism pinned to a screen edge** (Top Nav, Bottom Sheet, Toast, fixed footer CTA)
needs this in its spec. Missing it is invisible on a browser without a notch and broken on
the device your users actually hold.

## Touch: targets, spacing, and the hover trap

**Target size is a token, not a per-component guess.** WCAG 2.2 SC 2.5.8 requires 24x24 CSS
pixels at Level AA; SC 2.5.5 recommends 44x44 at AAA; Apple HIG specifies 44pt; Material
specifies 48dp. `build-architecture.md`'s component spec template uses 44x44 deliberately
above the AA floor, matching common mobile guidance. Treat 24x24 as the absolute floor for
a dense internal tool, not as a default to design toward.

**Spacing between targets counts.** SC 2.5.8 lets a target be smaller than 24x24 only if a
24px circle centered on it does not overlap any adjacent target. Dense icon rows need gaps,
not just big icons.

**Separate visual size from hit area.** A visually small icon button can still have a 44x44
hit area via padding or a pseudo-element, without changing how large it looks. This
resolves the tension between a dense visual language and an accessible touch target without
compromising either.

**Hover does not exist on touch, and hover-only interactions break.** A tooltip that only
appears on `:hover` is unreachable on a touch device: there is no hover state to trigger it,
only tap, which usually fires the underlying click instead. Gate hover-dependent
interactions behind a capability query rather than assuming a pointer:

```css
@media (hover: hover) and (pointer: fine) {
  .card:hover { box-shadow: var(--shadow-raised); }
}
```

**Any component whose only affordance is hover is broken on mobile by construction.**
Tooltip needs a tap-triggered alternative (or must not carry information found nowhere
else). Popover should already be tap-triggered per `naming.md`'s Tooltip vs Popover
distinction, which now has a second, mobile-specific reason to hold: Tooltip is
hover/focus-only and non-interactive by definition, so it is not just an accessibility
smell but a broken component on a touch device if it is the only way to reach information.

## Input quirks that must be baked into form components

**iOS Safari zooms into any input below 16px font-size**, and there is no viewport-meta
workaround that reliably disables it without also disabling pinch-zoom for the whole page
(a separate accessibility regression). The fix belongs in the component, not in tribal
knowledge: **every text input, textarea, and select must resolve to a 16px-or-larger font
size at the default root size.** Bake this into the Field molecule's spec so it cannot be
undershot per-instance.

**Set `inputmode` and `autocomplete` on every input by type.** `inputmode="numeric"` for a
quantity field, `inputmode="email"` for an email field, brings up the correct virtual
keyboard. `autocomplete` values (`email`, `tel`, `street-address`, `cc-number`) let the
browser and password manager fill correctly. Set `enterkeyhint` too (`search`, `send`, `next`, `done`) so the
return key says what it will do. Treat all three as required spec fields for the Text Input
and Number Input atoms, not optional polish.

**The virtual keyboard can cover a fixed-bottom element** (a CTA bar, a bottom sheet's
action row) when it opens. Chromium and Firefox support the viewport meta setting
`interactive-widget=resizes-content`, which makes the layout viewport shrink so fixed
elements move above the keyboard. **Safari does not support it**, so design fixed-bottom
components to survive being covered, and never put the only way to submit a form there. Test every fixed-bottom component with a real on-screen keyboard
open, not just in an emulator, since emulators do not reliably reproduce this.

**Scroll locking behind a modal is notoriously unreliable on iOS Safari** if you hand-roll
it with `overflow: hidden` on the body. Use the headless primitive's scroll lock (Radix
Dialog handles this) rather than writing your own, and pair it with
`overscroll-behavior: contain` on scrollable sheet content so the page behind does not
scroll through it.

## Components that must behave differently on mobile

Not a new component set. The same canonical inventory in `components.md`, with mobile
behavior specified as part of the same component rather than a fork:

| Component | Desktop | Mobile |
|---|---|---|
| Dialog | Centered overlay | Often full-screen or a Bottom Sheet, per product density |
| Select | Custom listbox | Consider the native `<select>` picker; it is faster and more familiar on touch, at the cost of custom styling |
| Tooltip | Hover-triggered | Must have a tap/focus path or must not gate unique information |
| Navigation | Persistent Side Nav or Top Nav | Hamburger menu or Bottom Nav tab bar, chosen by information architecture depth |
| Table | Full grid | Horizontal scroll with a frozen first column, or a stacked-card layout, chosen deliberately per data shape |
| Data Table | Full grid with all columns | Column priority: pick which 2-3 columns matter most, hide or drawer the rest |

Specify the mobile behavior in the same component's spec file (`build-architecture.md`
golden path), not as a separate component. A `MobileNav` sibling component is a naming
smell per the same logic that rejects slash names: it is the same concept with different
presentation, which is what a responsive spec is for.

## Performance: the mobile network and device tax

Core Web Vitals matter more on mobile because the network and CPU are both worse. Two
consequences for the design system specifically:

- **Image components must ship responsive sources** (`srcset`/`sizes`, or a Next.js/framework
  image component) by default, not as an opt-in. A hero image with no responsive sourcing
  is a guaranteed LCP regression, and heroes are already flagged as the common LCP culprit in
  `marketing-surfaces.md`.
- **INP (Interaction to Next Paint) suffers on low-end Android** specifically. Heavy
  client-side logic in an interactive component (a Combobox filtering a large list on every
  keystroke) should debounce and virtualize by default in the component itself, not be left
  to whoever consumes it to remember.

## Testing: emulation is necessary and insufficient

Configure the Storybook viewport addon with your **actual** breakpoints (see
`handoff.md`), not the defaults. Use Playwright's device emulation for CI-level coverage of
layout at real breakpoints.

**What emulation cannot catch, and needs a real device pass before shipping:** real touch
behavior (tap delay, momentum scroll), the virtual keyboard covering fixed elements, Safari
-specific bugs (the `100vh` issue existed for years before dvh existed to fix it, and iOS
Safari has its own history of scroll and viewport bugs), and the iOS input-zoom behavior
above, since a headless browser will not reproduce it.

## Recommended defaults

| Decision | Default |
|---|---|
| Breakpoint strategy | Mobile-first, content-driven, named not device-promised |
| Component layout queries | Container queries (`@container`) by default, viewport media queries for page-level only |
| Full-height layout | `dvh`, or `svh` for anything that must not be obscured by chrome |
| Safe areas | `env(safe-area-inset-*)` on every edge-pinned component, `viewport-fit=cover` set globally |
| Touch target | 44x44 minimum, 24x24 absolute floor with a documented reason |
| Hover interactions | Gated behind `(hover: hover) and (pointer: fine)`, with a non-hover path for any unique content |
| Input font size | 16px minimum at default root size, enforced in the Field spec |
| Select on mobile | Consider native, evaluate per product against styling needs |
| Testing | Storybook viewport addon on real breakpoints, Playwright emulation in CI, one real-device pass before ship |

## Contested

- **Native `<select>` vs a custom Listbox on mobile.** Native is faster, more familiar, and
  free of custom keyboard-handling bugs, at the cost of full style control. Reasonable
  systems land differently; decide per product rather than defaulting silently to custom.
- **Full-screen Dialog vs Bottom Sheet on mobile.** Both are legitimate; the choice should
  follow the product's information density and interaction depth, not be picked once for
  every dialog.
- **`dvh` vs `svh` as the default for full-height layout.** `dvh` tracks the real viewport
  and can cause layout jump as chrome shows/hides during scroll. `svh` is stable but can
  leave a visible gap when chrome is hidden. Pick per use case rather than standardizing one
  for the whole system.

- **Fluid body text.** Fixed `rem` body text is simpler and already user-scalable through
  the default font-size setting. Scaling it slightly with the viewport is defensible for
  editorial and marketing surfaces. Pick deliberately per surface type.

## Common failure modes

1. Designing on a 1440px desktop mock and treating "make it responsive" as a final pass
   rather than the default posture.
2. Using `100vh` for a modal or bottom sheet, which gets cut off by mobile browser chrome.
3. A tooltip carrying information available nowhere else, unreachable on touch.
4. An input styled below 16px, triggering an involuntary zoom on every focus on iOS.
5. A fixed bottom CTA bar with no safe-area padding, sitting under the home indicator.
6. Testing only in an emulator and shipping a real-device regression (input zoom, keyboard
   covering a button) that no CI check caught.
7. Building a second, forked "mobile" component instead of specifying responsive behavior
   in the one component's spec.
8. Making the whole spacing scale fluid, so components are a different size on every
   device.
9. Tying compact density to viewport width rather than pointer precision, which makes a
   touch tablet in landscape dense and hard to tap.
