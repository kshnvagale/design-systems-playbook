# Discovery: what to establish before recommending anything

Do not propose a token architecture, a component list, or a tooling stack before you can
answer these. Every question here exists because **the answer changes a recommendation**.
If an answer would not change what you do, it is not on this list.

Ask them in one batch, not one at a time. Offer the default so the user can say "yes,
default" instead of writing an essay.

---

There are two kinds of intake and you need both. **Part A** establishes constraints and
organization. **Part B** establishes what the thing should actually look like. Skipping
Part B produces a technically correct system that looks like every other system, which is
the most common way a from-scratch build disappoints the people who commissioned it.

---

# Part A: the eight questions that change the architecture

### 1. What kind of product, and how many?

Ask: *B2B, B2C, internal tooling, or a mix? How many distinct products will consume
this?*

| Answer | What it changes |
|---|---|
| Single B2C app | Lean component set, marketing surfaces matter, motion and personality budget |
| Single B2B/admin | Data table, filters, bulk actions, dense forms, every async state, keyboard nav |
| Internal tools | Speed over polish, admin CRUD patterns, skip marketing tier entirely |
| Mix of two or more | Shared core plus per-product composition. Not three systems, not one pretending they are identical |

Red flag: "all three, and they must look identical." They should share tokens and
primitives, not density and tone. Push back.

### 2. One brand, or many?

Ask: *Will this ever need to render under another company's branding (white-label,
partner, acquired product)?*

- **One brand:** neutrals can be brand-tinted, token count stays low, skip the theme
  abstraction.
- **Multi-brand or white-label:** true-neutral greys, brand differences live in the
  primitive layer only, semantic names stay identical across brands. Also check the
  mode-count ceiling of your design tool before designing around it.

This is the single most expensive thing to retrofit. Ask it early even if the answer
seems obviously "one."

### 3. Which platforms?

Ask: *Web only? Also native iOS/Android? React Native? Email?*

Web-only lets you keep tokens as CSS custom properties and stop. Anything cross-platform
forces a platform-neutral token source (DTCG JSON compiled per platform) and a naming
scheme that is not CSS-specific. Do not build cross-platform token infrastructure for a
web-only product.

### 4. Greenfield, or is there an existing product?

Ask: *Is there a live product today? Is there an existing design system or component
library, even a bad one?*

- **Existing product, no system:** start with an interface inventory of real screens.
  This is the normal case and the best one.
- **Existing system being replaced:** you need a migration and deprecation plan before
  you need new components, plus usage instrumentation to know what is actually used.
- **True greenfield, no product:** you have no evidence base. Build the smallest possible
  set from the first real screens being designed, and expect to be wrong. Do not
  architect a full system speculatively.

### 5. Who maintains it, and is that funded?

Ask: *Who owns this after launch? Is that their actual job, or on top of their job?*

The answer determines whether this survives. Central capacity is non-optional (see
`governance.md`). If the honest answer is "nobody, part-time," say plainly that the
system will stall, and scope to something a part-time owner can sustain: fewer
components, heavier reliance on a headless library, no custom tooling.

Red flag: "a federated group of volunteers from each team." That model has no track
record of working without central staffing.

### 6. What accessibility bar applies, and is it a legal requirement?

Ask: *WCAG 2.1 AA? 2.2? Any regulated context (government, health, finance, EU
accessibility act)?*

This decides whether contrast is a guideline or a gate, whether you need a high-contrast
theme as a separate axis, and how much of the budget goes to accessible primitives. In a
regulated context, ship WCAG 2.x compliance and treat APCA as informational only.

### 7. Who and what writes the UI code?

Ask: *Humans, AI agents, or both? Which agents and which stack?*

This is the question the pre-2026 literature does not ask, and it changes the delivery
mechanism entirely. If agents write a meaningful share of the code, you need a machine
consumable interface (registry, MCP, structured specs) and compile-time enforcement, not
documentation. See `ai-agents.md`.

Also ask: *Tailwind, CSS-in-JS, vanilla CSS, something else?* If Tailwind, aligning
primitive-tier naming to its conventions is a legitimate default because it reduces the
model's fight with its own training prior.

### 8. What already exists that you are not allowed to change?

Ask: *Locked brand colors? A mandated font? An existing marketing site? A component
library already in the codebase?*

Most systems are constrained before they start. The common one: a brand color that fails
contrast as an interactive foreground. Surface that in discovery, not after the palette
is built, because the fix (brand hue for large fills, a different accessible step for
interactive text) is an architectural decision, not a tweak.

---

---

# Part B: the questions that determine how it looks

Architecture decisions do not produce a look. If you only run Part A you will ship a
correct, generic system. Ask these in the same batch.

### 9. What is the primary brand color, and is it fixed?

Ask: *What is your primary brand color (hex)? Is it locked by brand, or can it be
adjusted for product use?*

Then **immediately check it as an interactive foreground on white and report the result in
the same reply.** Do not wait until palette construction. If it fails 4.5:1, that is an
architectural finding, not a detail, and the user needs to hear it before they get
attached to a mockup.

The standard resolution when it fails, which you should offer in the same breath so it
does not read as a rejection of their brand:

- Keep the literal brand hue for large fills, logos, illustration, and marketing surfaces.
- Derive a darker, higher-chroma step from the same hue family as the accessible
  interactive color for text, icons, and links.
- Both live in the same ramp, so it still reads as one brand.

Bright yellows, limes, oranges, and mid-greens fail this constantly. Assume you will need
this conversation and prepare it rather than discovering it late.

Also ask whether there are **secondary or accent brand colors**, and whether any color
already carries fixed meaning in the business (a specific green for approved, a specific
red for declined). Those constrain your semantic palette before you start.

### 10. Show me what you want it to feel like

Ask for artifacts, not adjectives:

> Send me 2-3 products whose look and feel you want to be in the neighborhood of, and 1
> you specifically do not want to look like. Screenshots or links are better than
> descriptions. If you have a moodboard, brand guidelines, a marketing site, or existing
> app screens, send those too.

"Modern and clean" means nothing and everyone says it. A screenshot resolves in seconds
what a paragraph of adjectives cannot. The negative example is as useful as the positive
ones and people find it easier to answer.

**Distinguish two different kinds of image and treat them differently:**

| Artifact | What it is | What to do with it |
|---|---|---|
| Screens of *their existing product* | Evidence of what is | Interface inventory: what components actually recur, what the real density and content lengths are |
| Reference products / moodboard | Aspiration, what should be | Extract the visual language: radius, density, elevation, type scale, neutral temperature |

Never confuse them. Building toward the moodboard while ignoring their real content
lengths produces a system that breaks on contact with actual data.

### 11. What to actually extract from the references

A moodboard is useless unless you know what to read from it. Do not describe the vibe.
Extract these, and state what you extracted so the user can correct you:

| Read this | Produces this decision |
|---|---|
| Corner radius language (sharp, soft, fully rounded) and whether actions and containers differ | Radius scale and its rules |
| Whitespace between and inside elements | Spacing scale coarseness, default density |
| Shadow usage vs borders vs background steps for separation | Whether elevation is a real axis or borders do the work |
| Type scale range and weight contrast | Number of type roles, whether weight or color carries emphasis |
| Neutral temperature (warm grey, cool grey, true neutral) | Neutral ramp construction, and whether it is safe for multi-brand |
| How much color appears, and on what | Whether brand color means interactive or is decorative |
| Border presence and weight | Border token count and whether meaningful/decorative split is needed |
| What has the most visual weight on a busy screen | What the system should optimize for |

That last row is the most useful question you can ask about any reference: **what is the
hero here, and what deliberately gets out of the way?** A system where the content grid
is the hero looks nothing like one where the chrome is, even with identical tokens.

### 12. Personality, as forced choices

If they cannot supply references, do not ask an open question. Offer opposed pairs and
ask them to pick a point between each. This takes a user thirty seconds and gives you
more than a paragraph of description would:

- Dense and information-rich  <->  airy and spacious
- Sharp corners  <->  fully rounded
- Flat, separated by borders  <->  layered, separated by shadow
- Restrained and neutral  <->  expressive and colorful
- Warm neutrals  <->  cool neutrals
- Serious and institutional  <->  friendly and approachable

Then state how each answer converts into a token decision, so the choices feel
consequential rather than decorative.

### 13. Motion: usability, delight, or both?

The cleanest framing, and the one to put to the user directly: **motion as usability**
(orienting, showing what changed, confirming) versus **motion as delight** (character,
celebration, brand expression). Every product needs the first. The second is a choice,
and it is expensive to add convincingly later.

Ask: *Should this feel utilitarian and quick, or expressive and characterful? Name a
product whose motion you like.*

Material 3's two motion schemes are a useful way to make the question concrete because
they are genuinely opposed:

- **Standard**: eases into final values, minimal bounce, functional. For utilitarian
  products.
- **Expressive**: overshoots final values to add bounce, used for hero moments and key
  interactions.

Carbon draws the same line as **productive** versus **expressive** easing, and ships
different cubic-bezier curves for each. Either way, the decision is one axis, and you
should get an answer rather than defaulting silently.

For most B2B, fintech, and internal products the honest answer is standard/productive
with a small number of deliberate expressive moments. Say that out loud rather than
quietly shipping flat motion, because "no motion" is itself a decision users feel.

### 14. Where is the product allowed to have personality, and where is it banned?

This is the delight question, and asking it as a **budget with exclusions** gets a far
better answer than "how playful should it be?"

Ask: *Where should this feel human or celebratory? And where must it stay completely
plain?*

Delight lives in a small number of predictable places: first-run and onboarding, empty
states, success and completion moments, and small micro-interaction feedback. The
exclusions matter more, and in a fintech or healthcare context they are not negotiable:

- Never on an error, a failure, or a decline.
- Never on anything involving someone's money, health, or legal standing going wrong.
- Never on a destructive confirmation.

Dan Saffer's framing is the right discipline here: microinteractions are "an exercise in
restraint, in doing as much as possible with as little as possible." A useful test before
adding any: **name the uncertainty this moment creates for the user.** If you cannot name
it, the interaction is decoration.

Second rule worth agreeing up front: **feedback should be proportional to stakes.** A
destructive action warrants more weight than a reversible one. Mismatches erode trust in
both directions, whether that is a subtle animation on a high-stakes action or an
intrusive one on a trivial click.

### 15. Brand assets, starting with the logo

**Ask for the company logo explicitly, every time.** It is the one asset that is always
needed and almost never volunteered.

Ask: *Can you send your logo? SVG preferred. Do you have a horizontal lockup, a stacked
version, and a standalone mark or favicon? Is there a light-background and
dark-background version?*

Why it matters more than it sounds:

- It appears in the header, nav, empty states, loading screens, auth screens, and emails,
  which means several components depend on it before you have finished the token layer.
- The preview and Storybook both need it. A system reviewed without the logo gets reviewed
  as a generic kit rather than as their product, and stakeholders respond very differently
  to the two.
- It is a real constraint on the header: a wide horizontal lockup and a compact square mark
  produce different navigation layouts.
- **A logo that only exists in one color on one background will break in dark mode.** Find
  that out in discovery, not when the dark theme ships.
- The mark is often the honest source of the brand palette, and sometimes contradicts the
  hex someone quoted from memory.

If they have no logo yet, say so plainly and use a neutral placeholder wordmark, sized to
realistic lockup proportions so the layout does not have to change later.

While you are asking: *any other brand assets? Icon set, illustration library, photography
direction, brand guidelines PDF?*

Then the visual questions: *Do you have an illustration style or library? Photography,
illustration, 3D, or abstract? Is there an existing brand expression you must match?*

This matters because empty states, onboarding, error pages, and marketing surfaces all
need it, and if it is undecided each team invents their own. It is one of the fastest ways
a product starts looking like several products.

Also worth asking explicitly: **where is the brand allowed to be loud, and where should it
be invisible?** Most good products are visually quiet in the working surfaces and
expressive at the edges (onboarding, empty states, marketing, celebration). Getting that
boundary stated saves a lot of argument later.

### 16. Depth: borders or shadows?

Ask: *Should surfaces separate with borders and background steps, or with elevation and
shadow?*

Small question, large consequence. It decides whether elevation is a real token axis or a
near-empty one, and it is highly visible. It also interacts with dark mode, where shadows
barely register and lightness has to carry elevation instead.

If they cannot answer, infer it from their reference screenshots and state what you
inferred.

### 17. Sound and haptics

Only relevant for mobile or native. Ask whether either is in scope. Both are real design
system concerns with tokens of their own, and both are usually forgotten until an engineer
asks. If out of scope, say so explicitly so it is a decision rather than an oversight.

### 18. Typography

Ask: *Is there a brand typeface? Is it licensed for web and app use?*

Brand typefaces are frequently custom or licensed for print and marketing only, and
cannot legally ship in a product. Establish this in discovery, not after a type scale is
built on it. Have a substitute direction ready (a well-matched open-source face) and
confirm the fallback explicitly rather than silently substituting.

Also ask whether they need a separate display or numeric face. Finance, data, and
dashboard products often want tabular figures, which is a real typeface requirement, not
a preference.

---

## Write the visual philosophy down, in prose

The output of Part B is not just token values. It is **three to five sentences of visual
philosophy** that go into the system's own documentation and its agent-facing rules file.

This matters because mechanical rules alone produce generic output. Diana Wolosin's
distinction, from building a design-system MCP at Indeed, is the clearest framing: some
of what a system encodes is **correctness** (right component, spacing follows the rules),
which is mechanically checkable, and some is **judgment** (does this layout breathe, does
it feel like this specific product), which has to be reduced to something specific enough
for a model before it can be encoded at all. Token values encode the first. Only written
philosophy encodes the second.

Good philosophy statements are specific and falsifiable, and each one rules something
out. Shape them like these:

- "Brand color means interactive. If something is brand-colored, the user can act on it.
  Never use it as a passive background."
- "Pills for actions, soft rectangles for content. This is the strongest visual signature;
  getting it wrong makes the product look generic."
- "Separate with borders and background steps, not shadows. If you want a shadow, you
  probably want a raised surface plus a subtle border."
- "Type weight carries emphasis, not color. Secondary and tertiary text express hierarchy,
  not importance."
- "The content grid is the hero. Chrome gets out of the way and nothing decorative
  competes with it."

Bad ones are unfalsifiable: "clean, modern, and user-friendly" rules nothing out and
changes no decision.

Test each statement by asking: **could someone violate this?** If not, it is not a rule.

---

## Secondary questions, ask only if relevant

- **Dark mode required, or nice to have?** Design the semantic layer for it regardless.
  Only the authoring effort is optional.
- **Density modes needed?** Usually only enterprise/data-heavy products. Do not build
  the axis speculatively.
- **Data visualization?** If yes, it needs a separate palette with its own constraints.
  Ask early; it is regularly forgotten and then improvised per chart.
- **Internationalization / RTL?** Affects spacing token naming (logical vs physical
  properties) and component layout assumptions.
- **Existing analytics on component usage?** If yes, that is a better evidence base than
  an inventory. If no, plan to add it.

---

## Defaults to assume when you get no answer

Do not block on discovery. State the assumption and proceed.

| Unknown | Assume |
|---|---|
| Platform | Web only |
| Brands | Single brand |
| Accessibility bar | WCAG 2.2 AA, non-regulated |
| Dark mode | Required eventually, so semantic layer designed for it |
| Stack | React plus Tailwind, headless primitives underneath |
| Who writes code | Both humans and agents |
| Maintainers | One to two people, part-time, until proven otherwise |
| Token count target | 40-80 primitives, 60-120 semantic |
| Visual direction | Restrained, border-separated, medium density. State this and ask for a reference to correct it. |
| Brand color role | Interactive only, never decorative fill |
| Typeface | System stack or a well-supported open-source face until told otherwise |

Always say which assumptions you made. A wrong assumption stated out loud gets corrected
in one line; a wrong assumption left silent gets discovered three weeks in.

---

## What not to ask

Asking too much is its own failure. These are decisions to make, not questions to pose:

- **"How many steps should the color ramp have?"** Decide: 10-12 with a per-step purpose.
- **"What should we name tokens?"** Decide: general to specific, semantic tier by
  purpose. Show the convention, do not negotiate it.
- **"Modal or Dialog?"** Decide from ARIA.
- **"Should we use a headless library?"** Yes, unless there is a stated constraint.
- **"Figma or code as source of truth?"** Decide: code or a neutral spec, with the design
  tool as a view onto it.
- **"Which components should we build?"** Derive from inventory plus prevalence data. Do
  not ask the user to list them from memory; that produces a wish list, not evidence.

The distinction: ask about **context you cannot observe** (constraints, org, funding,
platform, legal). Decide anything that is a matter of craft, and explain the reasoning.
A user who wanted to make every craft decision themselves would not be asking.

---

## Red flags to name explicitly when you hear them

| What they say | What to say back |
|---|---|
| "We want it to do everything Material does" | Scope to your inventory. A generic component list produces a generic product. |
| "Every team will contribute" | Contribution needs central capacity first, and most teams never contribute. |
| "We need it done in two weeks" | Scope to tokens plus 10-15 components, or it will be abandoned. |
| "Just copy [competitor]'s system" | Their tokens carry their bugs, including contrast failures. Harvest usage, do not clone. |
| "Design system first, then we will build products with it" | Backwards. Build the system from screens you are already shipping. |
| "We will add dark mode later" | Fine to author later, but the semantic layer must assume it now or you re-touch every state. |
| "Let's use AI to generate the whole system" | Generation is fine. The enforcement layer is what makes it hold, and that is deterministic. |

---

## Output of discovery

Before proceeding, write back a short statement the user can correct in one pass:

1. Product type(s) and how many products
2. Brand and platform scope
3. Who maintains it and at what capacity
4. Accessibility bar
5. Who and what writes the UI code
6. Fixed constraints you must design around
7. **Primary color, and whether it passes as an interactive foreground**
8. **The visual direction in one line, plus what you read from their references**
9. **Brand assets received or missing**, logo explicitly, with any dark-background gap flagged
10. Explicit assumptions you made for anything unanswered
11. What you are going to do first, and what you are deliberately not doing yet

If they gave you references, say what you extracted ("soft 12px radii, generous
whitespace, borders rather than shadows, cool neutrals"). That is the fastest way to find
out you read the moodboard the way they meant it, and it costs one sentence.

If the user corrects one line, you have saved weeks. This step costs one message.
