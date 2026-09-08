# Marketing surfaces: landing pages, and the brand/product seam

The rest of this skill is oriented toward product UI and internal tools: dense screens,
forms, tables, restraint. Marketing surfaces follow different rules, and a system built
only for product will fail at them in predictable ways. This file covers the difference.

Read it if the scope includes a marketing site, landing pages, campaign pages, or a
pricing page. Skip it for a purely authenticated product.

---

## The decision: not one system, not two. Layers.

**Default: one shared primitive layer, with two semantic layers and two component sets on
top of it.**

The framing that works best comes from Indeed's brand systems team, who describe brand and
design system as **layers of the same surface** rather than as separate systems
([Indeed Design](https://indeed.design/article/meeting-the-moment-with-a-brand-system)):

| Layer | Owner | Character | Contains |
|---|---|---|---|
| **Brand content** | Brand system | Expressive | Campaign and marketing content, video, photography, illustration |
| **Structural elements** | Shared | Mixed | Product surfaces and interactive modules: type, color, size, spacing |
| **Foundation** | Design system | Functional | Core components and navigation: header, footer, menus, overlays |

The middle layer is the interesting one. It is jointly owned, and it is where most of the
argument happens.

A second useful model treats a corporate brand system as sitting **above** several
touchpoint-specific sub-systems (website, product, internal tools), so that changing a
brand value in one place propagates to all of them
([DPDK, Scaling Design](https://6265534.fs1.hubspotusercontent-na1.net/hubfs/6265534/Marketing%20-%20Files/Scaling%20design%20guide/DPDK_Scaling%20design.pdf)).
Same idea, different diagram: shared root, divergent branches.

**What flips the default to genuinely separate systems:** a marketing site on a different
stack that shares no code with the product (a CMS-driven marketing site next to a React
app), or a brand team organizationally separate from product with no shared roadmap. That
second one is common. At Productboard, for example, brand designers sit under marketing and
own the website, social, ebooks, and events, entirely outside the product design system
([Productboard](https://www.productboard.com/blog/design-at-productboard-how-we-work-2-3)).
When ownership is split that cleanly, forcing one system creates friction without
delivering coherence. Share tokens, accept two component sets.

---

## What actually differs

| Dimension | Product UI | Marketing surfaces |
|---|---|---|
| **Type scale** | Tops out around 32-40px | Needs 56-120px+ display sizes |
| **Spacing** | 4-24px, tight, density matters | 64-160px section rhythm |
| **Layout** | Constrained containers, fixed chrome | Full-bleed, asymmetric, per-page bespoke |
| **Palette** | Neutral-dominant, brand reserved for action | Brand-forward, large saturated fields |
| **Motion** | Functional, 50-400ms, clarifying | Expressive, scroll-linked, longer, decorative is allowed |
| **Imagery** | Sparse, functional | Central. Photography, illustration, video |
| **Components** | Reused hundreds of times | Often used once, on one page |
| **Content** | Real user data, unpredictable lengths | Authored copy, controlled lengths |
| **Success metric** | Task completion, error rate | Conversion, scroll depth, bounce |
| **Hard constraints** | Accessibility, keyboard, density | Accessibility, **plus Core Web Vitals and SEO** |

The two constraints on the right that have no product equivalent are worth calling out.
**LCP is usually the hero image**, so a marketing system that ships an unoptimized hero
component is shipping a performance regression on the most-measured page the company has.
And **semantic heading structure matters for SEO** in a way it does not inside an
authenticated app, which raises the stakes on the semantic-heading gate in
`build-architecture.md`.

---

## Token architecture that serves both

**Shared, no exceptions:**

- The primitive layer. One brand hue, one neutral ramp, one set of status colors. If
  marketing and product resolve `brand.primary` to different values, the brand is broken
  and no amount of component work fixes it.
- The icon set.
- Border radius language.
- Voice and tone (see `visual-language.md`).

**Extended for marketing, not replaced:**

- **Type scale.** Product needs roughly 8-12 roles topping out near 40px. Marketing adds
  display roles above that: `display.sm`, `display.md`, `display.lg`, `display.xl`. Same
  family, same ratio, extended upward. Do not fork the scale; extend it.
- **Spacing.** Add section-level steps (`space.64`, `space.96`, `space.128`, `space.160`)
  above the product scale. Same base unit, longer tail.
- **Motion.** This is where the productive/expressive split in `visual-language.md` earns
  its keep. Marketing uses the expressive scheme, product uses productive. **Two named
  schemes, one token structure.** That is how you let marketing be expressive without
  expressive motion leaking into a data table.
- **Semantic color.** Marketing needs a few tokens product does not: large brand fields,
  gradient stops if the brand uses them, on-image text colors, and scrim values.

**The rule that keeps it honest:** marketing extends the scale, it never redefines a token
that product already uses. If marketing needs `space.24` to mean something different, that
is a fork, and you now have two systems whether you admit it or not.

---

## When marketing work happens in the sequence

Two different answers, because tokens and components are on different clocks.

**Token extensions happen in the token phase, with the primitives.** If marketing is in
scope at discovery, extend the type and spacing scales in step 4 alongside everything else.
Do not defer them. The scales are one artifact, and extending them later means re-running
the token build, re-checking contrast on the new steps, and re-authoring anything that was
built against the shorter scale. It costs almost nothing to add display roles and section
spacing while you are already there.

**Marketing components happen after the product system is approved**, as a distinct phase
with its own preview and its own gate. They are organisms composed from approved atoms and
molecules, so they cannot be built before those exist.

The sequence:

```
step 4  tokens        primitives + product scales + marketing scale EXTENSIONS
step 7  preview 1     base product system            -> gate 1
step 7a preview 2     page templates (if wanted)     -> gate 2
step 7b preview 3     marketing surfaces (if in scope) -> gate 3
```

**If marketing was not in scope at discovery and arrives later**, extend the existing
scales rather than forking them. That is a retrofit, and it is mildly annoying. Forking is
not annoying, it is permanent.

## Separate or merged: keep them separate, sharing a core

**Separate package or directory, importing from the shared core.** Not merged into the
product component set.

```
packages/
  tokens/           shared primitives, product scales, marketing extensions
  ui-core/          Button, Link, Icon, Card, Input, Accordion, Grid, Stack
  ui-product/       product organisms: DataTable, CommandPalette, AppShell
  ui-marketing/     Hero, PricingTable, LogoWall, CTABand, MarketingFooter
```

Four reasons this beats merging:

1. **Bundle.** The product app should not ship a Pricing Table. Marketing should not ship
   a Data Table. Separate packages make that automatic rather than a tree-shaking hope.
2. **Dependency direction stays legible.** Both import from `ui-core`; neither imports the
   other. That is enforceable with the boundary audit in `build-architecture.md`. A merged
   set has no such line to draw.
3. **Different review bars.** A marketing component is often used once and can ship at 90%.
   A product component is used in 200 places and cannot. Merging averages the two bars,
   usually downward.
4. **Different owners in most orgs.** When brand sits under marketing, separate packages
   mean separate ownership without separate tokens.

**What must not be separate:** the token layer and `ui-core`. One brand hue, one neutral
ramp, one icon set, one Button. The moment marketing has its own Button, the seam at
sign-up becomes visible and no amount of token sharing hides it.

## Marketing component inventory

An **addition** to the canonical inventory in `components.md`, not a replacement. Same
atomic vocabulary.

**Marketing organisms (12):**

Hero · Feature Grid · Pricing Table · Testimonial · Logo Wall · Stat Band · CTA Band ·
FAQ (accordion-based) · Marketing Footer · Marketing Nav (mega menu) · Newsletter Capture ·
Content Card Grid (blog, case studies, resources)

**Marketing molecules (6):**

Eyebrow + Headline + Subhead lockup · Feature Item (icon, title, body) · Plan Card ·
Quote Block · Media + Text Split · Section Header

For comparison, the U.S. Web Design System's landing page template composes just six
things: extended header, hero, graphic list, button, grid, and medium footer
([USWDS](https://designsystem.digital.gov/templates/landing-page)). A landing page needs
fewer distinct components than people expect. The variety comes from composition and
content, not from component count.

**Genuinely shared with product:** Button, Link, Icon, Badge, Card, Accordion, Tabs, Input
(newsletter, contact), Avatar, Divider, Grid, Stack, Container.

**Never shared:** anything with product state. A marketing page has no loading state for a
data table, no permission-aware disabled button, no empty state for a real user's cart.

---

## Rules that keep the two coherent

1. **One brand color meaning.** If brand green means "interactive" in product, it cannot
   mean "decorative background" in marketing. Pick one, document it in the visual
   philosophy, apply it to both.
2. **One icon set.** Two icon sets is the fastest visible tell that a company runs two
   systems.
3. **No seam at the boundary.** The most-crossed transition is the marketing site into the
   app, usually via sign-up. Design that handoff deliberately: the same header treatment,
   the same button, the same type. A user should not feel the software change hands.
4. **Marketing components live in a separate package or directory**, importing from the
   shared core. This keeps the product bundle free of a Pricing Table nobody ships in the
   app, and makes the dependency direction explicit (see `build-architecture.md`).
5. **The product system does not take marketing's constraints.** Do not add display type
   or 160px spacing to the product scale because marketing needed it. Extend in the
   marketing layer.

---

## What to ask in discovery

Add these when marketing is in scope (see `discovery.md`):

- Is there a marketing site today, and is it on the same stack as the product?
- Who owns it? Same team, or a brand team under marketing?
- Which pages: home, pricing, product pages, blog, case studies, careers, legal?
- Is there an existing brand identity or brand guidelines document the site must follow?
- Is the site CMS-driven? If so, components need a content schema, not just props.
- Are there Core Web Vitals or SEO targets someone is accountable for?

That third question matters more than it looks. **A pricing page and a blog index are
different systems' worth of work**, and teams routinely say "a marketing site" meaning
one landing page or meaning forty templates.

---

## Contested

- **One system or two.** Genuinely unsettled and mostly determined by org structure rather
  than by design theory. The layered model works when brand and product share a roadmap.
  Where brand sits under marketing with separate goals, two systems sharing a token layer
  is the honest answer, and pretending otherwise produces a system neither team uses.
- **Whether marketing components belong in the design system at all.** A defensible
  position is that they are per-campaign, live in the marketing site's repo, and should
  never be systematized because their reuse rate is genuinely low. The counter is that
  Hero, Pricing Table, and Footer get rebuilt every quarter regardless, so they may as
  well be built once.
- **How much motion is too much.** Product norms say motion must clarify. Marketing norms
  allow motion that exists purely to delight or to hold attention. Both are correct in
  their context, which is why the two motion schemes must be named and separate.

---

## Common failure modes

1. **Using the product type scale for a landing page.** The hero tops out at 40px, the page
   reads as an admin screen, and someone adds an arbitrary `text-[72px]` that bypasses the
   whole system.
2. **Using the product spacing scale for section rhythm.** Sections at 24px gaps read as a
   dense form rather than a marketing page.
3. **Forking the palette for marketing** because the product palette felt too restrained.
   Now brand color means two things and neither team can change it safely.
4. **Building marketing components into the product package**, shipping a Pricing Table in
   the app bundle.
5. **Letting expressive motion leak into product.** Scroll-linked parallax is fine on a
   landing page and actively hostile in a data table.
6. **Ignoring LCP on the hero.** The most visible page in the company ships a performance
   regression that nobody in the design system team is watching.
7. **A visible seam at sign-up**, where the marketing site and the app clearly disagree
   about what a button looks like.
