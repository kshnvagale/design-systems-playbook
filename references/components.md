# Component selection, build order, and API design

Governance, adoption, and team models live in `governance.md`. This file
is about what to build and how to shape it.

## The strategic point most teams miss

Nathan Curtis, asked in 2026 where the community's blind spot is:

> "While essential to a system's foundation, our community is **focused too much on
> design tokens**. Every day people are posting some design token tool they built or
> article they wrote that solves the same design token challenge in a subtly different
> way. In the projects I'm participating in, those token challenges are already solved.
> The work ahead of me is crafting components well, including structure, props, styling,
> behavior, motion and accessibility. **Component architecture is an order of magnitude
> more complicated than tokens.** Far fewer systems-interested people seem to be talking
> about that."
> ([Design Systems Collective](https://www.designsystemscollective.com/were-focused-too-much-on-design-tokens-nathan-curtis-on-design-systems-today-a329fdd79d4c))

Budget your time accordingly. Tokens are a largely solved problem with well-trodden
answers (see `tokens.md`). Component API design is where the genuine
difficulty and the genuine differentiation are.

## How to decide what to build: interface inventory, not a shopping list

Brad Frost's **Interface Inventory** method is the standard first step: gather a
cross-disciplinary group (designers, engineers, PM, QA, not just design), screenshot
every unique UI pattern across the real live product, and categorize. Suggested starting
categories: Global (headers/footers), Navigation, Image types, Icons, Lists, Interactive
Components, Media, 3rd Party, Messaging (alerts/errors/validation/tooltips), Colors.
Time-box the screenshotting pass; 30-90 minutes is usually enough for a first pass.
Source: [bradfrost.com](https://bradfrost.com/blog/post/conducting-an-interface-inventory/).

The output is not the design system. It is the evidence base that justifies one, and it
surfaces that "designers, developers, and product owners often have different names for
the same UI pattern," which is exactly the naming work that has to happen next.

**Do this before opening any existing component library for inspiration.** Building from
your actual product's screens rather than cloning Material's or Ant's component list is
what prevents building 40 components nobody uses while missing the 3 weird ones your
product genuinely needs.

Concrete proof this matters: Blade's roughly 60-component library contains `Amount` (a
currency-formatting-aware component), `TrustBadge`, `RazorSense`, and
`SpotlightPopoverTour`. No generic library would have suggested any of those. They exist
because Razorpay is a payments company and the inventory said so.

## Prevalence data: what recurs across real systems

[Component Gallery](https://component.gallery/components/) indexes components across
published design systems and reports how many include each one. This is real evidence
for prioritization rather than a guess. Full ranked data:

| Component | Systems | Component | Systems |
|---|---|---|---|
| Badge / Tag / Chip | **123** | Popover | 50 |
| Button | **118** | Pagination | 48 |
| Alert | **108** | Icon | 45 |
| Accordion | **101** | Datepicker | 44 |
| Radio button | 85 | Toast | 41 |
| Checkbox | 84 | Progress bar | 40 |
| Modal / Dialog | 82 | Slider | 38 |
| Select | 82 | Avatar | 38 |
| Tabs | 80 | Drawer | 38 |
| Card | 77 | Header | 38 |
| Tooltip | 74 | Progress indicator | 38 |
| Table | 73 | Combobox | 37 |
| Text input | 72 | Button group | 35 |
| List | 67 | Separator | 34 |
| Spinner | 66 | Fieldset | 32 |
| Link | 64 | File upload | 32 |
| Navigation | 62 | Skeleton | 30 |
| Toggle / Switch | 60 | Search input | 30 |
| Breadcrumbs | 55 | Heading | 29 |
| Textarea | 52 | Segmented control | 28 |

Long tail (build only on evidence): Carousel 22, Form 21, Stepper 20, Rating 19,
Footer 19, Color picker 18, Date input 18, Empty state 16, Label 15, Video 15,
Skip link 14, Tree view 14, Quote 11, Visually hidden 11, Stack 10, Hero 9, File 6,
Rich text editor 5.

Three things worth noticing, because they contradict common intuition:

1. **Badge is the single most common component (123), ahead of Button.** Almost every
   system needs small metadata labels, and almost every team underestimates this.
2. **Accordion (101) outranks Card (77) and Table (73).** It usually gets treated as a
   tier-2 component; the data says it is close to universal.
3. **Rich text editor appears in only 5 systems, Color picker in 18.** These are the ones
   teams keep proposing early. The evidence says almost nobody puts them in a shared
   system.

Treat this as prevalence, not prescription. It tells you what is *usually* needed. Your
interface inventory tells you what *you* need. Where they disagree, your inventory wins.

## The canonical component inventory

Prevalence data tells you what recurs. This tells you what your system needs to **contain**
by v1. It exists because the most common real failure is not building the wrong component,
it is silently omitting whole ones: tab groups, side drawers, switches, toggles, and
accordions all go missing when the inventory lives only in someone's head.

Treat this as the **floor**. Domain components go on top.

### Atoms (24)

Button · Icon Button · Link · Text · Heading · Icon · Badge · Status Dot · Avatar ·
Divider · Spinner · Skeleton · Checkbox · Radio · **Switch** · **Toggle Button** ·
Text Input · Text Area · Number Input · Select · Slider · Kbd · Code · Progress Bar

### Molecules (27)

Field (label + input + help + error) · Button Group · **Toggle Button Group** ·
Segmented Control · Avatar Group · Breadcrumbs · Pagination · **Tab List / Tab Group** ·
**Accordion (Collapsible)** · Card · Clickable Card · Selectable Card · Alert · Banner ·
Toast · Tooltip · Popover · Hover Card · Dropdown Menu · More Menu (overflow) ·
Search Input · Stepper · Empty State · Chip / Tag · Date Input · Timestamp · Metadata List

### Organisms (16)

App Shell · Top Nav · Side Nav · **Side Drawer** · **Bottom Sheet** · Dialog / Modal ·
Command Palette · Table · Data Table · List · Tree List · Toolbar · Carousel ·
Calendar / Date Picker · File Upload · Typeahead / Combobox

### Layout primitives (6)

Stack · Grid · Section · Container · Aspect Ratio · Resize Handle

### Where this comes from

Derived from prevalence data above plus **Meta's Astryx**
([astryx.atmeta.com/components](https://astryx.atmeta.com/components)), a real shipping
system with roughly 90 components across Action, Chat, Container, Content, Data Input,
Feedback and Status, Layout, Navigation, Overlay, Table and List, and Utility.

Two transferable lessons from how Astryx is organized:

**It categorizes by function, not composition.** Action, Overlay, Data Input, Navigation.
Atomic layers organize by composition instead. These are not in conflict and you want both:
functional grouping for browsing and documentation, atomic layers for build order and
dependency direction. Same components, two indexes.

**It ships a whole Chat cluster** (Chat Composer, Chat Message, Chat Tool Calls, Chat
System Message) because Meta builds AI products. That is the clearest possible evidence for
inventory-driven building: nothing generic would suggest those, and they exist because the
real product needed them. A payments company ships an `Amount`. A health product ships a
vitals display. **The canonical list is the minimum; your interface inventory supplies the
rest.**

### Inventory versus build order: do not confuse them

| Question | Answered by | Applies to |
|---|---|---|
| What must exist? | The canonical inventory | Completeness. The HTML preview shows all of it. |
| What gets built first? | The tiers below | Sequencing. React build order only. |

**Deferring a component in the build order is never permission to omit it from the
inventory or from the HTML preview.** Those are different artifacts answering different
questions. See `delivery.md`.

The layer names here are the same ones used in the HTML preview sections and in package
structure (`build-architecture.md`), so one vocabulary runs from preview to shipped code.

## A defensible build order

**Tier 1 (build first, roughly 12-16 components).** High-frequency, low-ambiguity, and
each has a matching WAI-ARIA APG pattern to build against rather than improvising:
Button, Text input, Checkbox, Radio, Select/Listbox, **Badge**, Card, Modal/Dialog,
Tabs, Tooltip, **Alert**, Link, Spinner, Icon Button, Divider/Separator, Avatar.

**Tier 2 (once tier 1 is stable and actually adopted).** Mostly compositions of tier-1
primitives: **Accordion**, Table (basic, sortable), Pagination, Breadcrumbs, Dropdown
Menu, Toast, Toggle/Switch, Progress bar, Skeleton, Empty state, Banner, Textarea, Form
field wrapper (label + input + error + helper text composed as one unit), Segmented
control, Popover, Drawer.

**Tier 3 (defer until real, repeated, cross-team demand exists).** The classic
time-sinks: **Data Table** (sorting, filtering, column resize, bulk selection,
virtualization), **Date Picker** (calendar math, ranges, timezones), **Combobox /
Autocomplete** (async search, keyboard nav, screen-reader announcements), **Rich Text
Editor**, **Charts / data-viz**, **File Upload** (drag-drop, progress, chunking,
validation). Each can consume a person-month or more and each has deep, non-obvious
accessibility requirements.

**Do not let a junior team build tier 3 from scratch as their first design-system work.**
This is precisely where headless primitive libraries earn their cost.

## B2B vs B2C vs internal tools

- **B2B / enterprise** needs disproportionately: dense data tables with filters and bulk
  actions, keyboard-navigable forms, explicit empty/loading/error states for every async
  view, and permission-aware states (disabled-with-a-reason, not just disabled). Density
  and information scent beat visual flourish. Users are in the product for hours daily.
- **B2C** needs: marketing and landing surfaces, imagery-heavy cards, conversion-flow
  components (cart, checkout steps, pricing tables), and more tolerance for motion and
  personality.
- **Internal tools** need: speed of assembly over pixel polish, tolerance for "ugly but
  fast," heavy reuse of admin CRUD patterns (list -> detail -> edit -> confirm delete).
  Usually skip the marketing tier entirely.

A company running all three at once is exactly Razorpay's situation with Blade. The
resolution used in production is a **shared core (tokens plus primitives) with
per-product theming and composition on top**, not three separate systems, and not one
system pretending the three contexts are identical. Blade supports genuine
white-labelling through a documented `createTheme` function and a live Theme Playground,
because Razorpay ships checkout UI that appears under different partner banks' branding.

## What belongs in the shared system at all

Use Brad Frost's component / recipe / snowflake split (detailed in
`governance.md`). Short version:

- **Component**: content- and context-agnostic, maximally reusable. Goes in the system.
- **Recipe**: a specific composition used consistently within one product. Lives in that
  product, or in a separate extras library.
- **Snowflake**: genuine one-off. Lives in the product.

Things get promoted *down* into the system after proving cross-team need. They do not
start there speculatively. A single team's loud request is not sufficient justification
and is the single most common way shared libraries bloat.

## Component API design

- **Compound components over monolithic prop-soup.** A `Card` taking `title`,
  `subtitle`, `imageUrl`, `footerText`, `footerActionLabel` as flat props does not
  scale. `Card` / `CardHeader` / `CardBody` / `CardFooter` does, because consumers can
  omit and reorder slots without the author having anticipated every combination.
- **Watch for the three specific complexity smells** Curtis names:
  1. Low-level props controlling internal elements (`footerSubtitleLabel`,
     `headerNotificationDotCount`).
  2. Overlapping props (a Badge with `appearance` *and* `emphasis` *and* `variant`),
     which shows up to designers as "missing cells in their variant grid."
  3. Style configuration props (a Card exposing `border`, `padding`, `cornerRadius`).

  All three are clear individually and overwhelming collectively, and all three push the
  component toward rigid structure.
- **Controlled and uncontrolled both.** An uncontrolled default with an optional
  controlled override (`value` / `onChange`) is the React-ecosystem convention and avoids
  forcing every consumer into state management for simple cases.
- **Composition over configuration, but not absolutely.** If two props are frequently
  used together and their combination has a name a designer would say out loud ("the
  destructive secondary button"), promote it to a real named variant. If the
  combinations are unbounded, you need slots, not more props.
- **Escape hatches are a genuine, contested tradeoff.** `className` / `style` / `asChild`
  give consumers a pressure valve so they do not fork the component, and are also exactly
  how visual drift creeps in. The defensible line: **allow content composition freely,
  block visual-prop overrides at the lint level.** If someone needs a different fill or
  spacing than the component offers, that is a signal the component is missing a variant,
  not that the override should be permitted.
- **Polymorphism (`as` / `asChild`)** is useful for "renders as a Link, looks like a
  Button" without duplicating styles onto a `LinkButton`. Useful in moderation; making
  everything polymorphic by default complicates typing and accessibility semantics for
  little gain.

### The composition pendulum, and where it is heading

Curtis, on getting early access to Figma's native slots and converting an existing
library:

> "It's remarkable that as a pendulum starts to swing from a prop-heavy supercomponent
> to leverage slots, the pendulum swings hard. The impact across a catalog is profound.
> **Props disappear. Subcomponents are being retired. The team leans much more into
> examples.**"

The consequence he flags is real and worth planning for: composition hands makers a
blank canvas, and they then need to know "What could I make here? What am I *allowed* to
make here? What works best?" The answer is to stock docs with **ready-made examples** as
starting points, for humans and AI agents alike, rather than relying on a rigid API to
prevent misuse.

He also predicts the organizational friction: "I'm not convinced how many system makers,
particularly design leaders, will be willing to relinquish the veneer of control they
think they have with rigid, strongly opinionated API."

## Headless primitives: use one, do not hand-roll accessible widgets

Accessible interaction patterns for Menu, Combobox, Dialog, and Tooltip involve dozens of
non-obvious details (focus trap ordering, escape handling, typeahead, ARIA live
announcements, portal and z-index management). They take specialist time to get right and
are easy to get silently wrong in ways only keyboard and screen-reader users hit. Building
these from scratch as v1 work is a common reason a first attempt ships broken
accessibility.

- **[Radix UI Primitives](https://www.radix-ui.com/primitives)** - unstyled accessible
  React primitives for exactly the hard components. The de facto default foundation for a
  from-scratch React system today.
- **React Aria (Adobe)** - hook-based, lower-level than Radix, powers Adobe's own React
  Spectrum. Strong if you want total styling control with no compromise on behavior.
  Adobe itself uses this split at enormous scale, which is the best available proof the
  architecture works.
- **Base UI, Ark UI, Headless UI** - same category, different API shapes. Evaluate on
  bundle size and ergonomics for your stack.
- **shadcn/ui's copy-paste registry model** is a distribution strategy worth understanding
  even if you do not adopt shadcn: component source is copied into your repo via CLI
  rather than installed as an npm dependency, so your team owns and can edit it from day
  one. Trades easy upstream updates for full ownership.

**Recommendation:** build tier-1 and tier-2 components as thin styled wrappers around a
headless library. Spend your engineering effort on the visual/token layer and the
composition API, not on reinventing focus-trap logic.

## Multi-product theming

Every multi-product system studied converges on the same shape: **one shared primitive
plus semantic token layer, one shared component layer, and product-specific composition
one level up.** Product teams assemble screens from shared components; they do not fork
the components. Brand differentiation lives in which values the semantic tokens resolve
to, not in duplicated component code.

## Contested

- **How permissive escape hatches should be.** Genuine ongoing tension; serious systems
  land in different places.
- **Headless-library dependency risk.** Some teams are wary of building core UX on a
  third-party primitive that could change API or be abandoned. The counter is that
  hand-rolled accessibility is the worse long-term risk, especially for Combobox and
  Dialog.
- **Props vs slots.** Currently mid-swing. Curtis expects a hard swing toward slots and
  composition; many design leaders will resist it. Know which side you are choosing and
  why.
- **Adopting an off-the-shelf base (MUI/Chakra/Ant) and theming it.** Legitimate and
  faster when your UI is not deeply differentiated. Curtis notes large enterprises he
  works with still do not favor Material/Fluent/shadcn over their own libraries, but
  increasingly adapt their architecture to conventions like Tailwind's.
