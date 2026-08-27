# Operating model, governance, and adoption

This is the part that decides whether the system survives. Token architecture and
component APIs are solvable engineering problems. Adoption is an organizational one,
and it is where most systems actually die.

## Start here: a design system is a product, not a project

Nathan Curtis's framing, and the single most load-bearing idea in this file: treat the
[design system as a product, serving products](https://medium.com/eightshapes-llc/a-design-system-isn-t-a-project-it-s-a-product-serving-products-74dcfffef935).

> "The strongest systems treat it as infrastructure and build it not to deliver but
> instead prepared to predictably change... That means operating with a vision,
> roadmap, backlog and incremental and smooth release process. Systems freeze and
> collapse with opposite traits like a vague mission, poor processes, uneven or
> inconsistent staffing commitment, undependable delivery of features over time, and
> inability to change in ways large and small."
> ([Design Systems Collective interview, 2026](https://www.designsystemscollective.com/were-focused-too-much-on-design-tokens-nathan-curtis-on-design-systems-today-a329fdd79d4c))

Practical test of whether you are running it as a product: can you name the system's
current roadmap, its last three releases, and who its users are? If not, it is a
project, and it will stall when the founding team gets reassigned.

## Team models, and the correction almost everyone missed

Curtis's [Team Models for Scaling a Design System](https://medium.com/eightshapes-llc/team-models-for-scaling-a-design-system-2cf9d03be6a0)
(2015) defined three models, and became the most-cited concept in the field:

1. **Solitary** ("overlord") - one person or team builds a system centred on their own
   needs. Low ceiling, high bus-factor risk.
2. **Centralized** - a dedicated team produces and supports the system as their actual
   job. Strong consistency; risk of drifting from real product needs.
3. **Federated** - designers/engineers from multiple product teams collectively decide
   and contribute, without a dedicated central team.

**Critical correction, and the reason this file exists:** in 2024 Curtis published
[The Fallacy of Federated Design Systems](https://medium.com/@nathanacurtis/the-fallacy-of-federated-design-systems-23b9a9a05542)
explicitly retracting the original article's tilt toward federated. Most secondary
writing on the internet still repeats the 2015 framing without the retraction. Do not
repeat it. His actual position now:

> "**Favoring federated was and is wrong.**... federated is not a choice, it's a facet.
> In practice, it's *never* pursued first and *never* without central investment."

Across 80+ design systems he consulted on, the number that succeeded without centrally
allocated people was, in his words, "Zero. Zero. Zero percent."

> "In my practice, successful design systems *always* have a central team and *always*
> seek the participation (and, if worth it, contribution) from a federated community."

**So the actual rule is: central first, always. Add federated contribution gradually,
as an optional facet, once a stable core exists.** Not "pick centralized or federated."
Not "hybrid" as a starting position either, because that still implies federated is
load-bearing on day one. It is not.

Curtis also names the specific stakeholder myths that positioning federated as a goal
anchors in people's heads. Expect to have to unwind all of these, and know that
unwinding them is politically risky work:

| Myth | Reality |
|---|---|
| Everyone can, should and will contribute | Few if any contribute meaningful new features |
| Every component made goes in the system | Almost all components made across teams do not belong in it |
| Anyone who makes UI is capable of contributing | Most contributors lack the systems experience to contribute efficiently |
| Components of any composition level are welcome | Core libraries are mostly lower-level UI components, deliberately |
| Federated contribution is our primary success indicator | Core libraries with zero outside contributions can be extremely valuable |
| Federated gets you a design system for free | It requires real capital and ongoing operational cost |

Atlassian, a system Curtis praises, [explicitly limits contributions to fixes only](https://atlassian.design/resources/contribution),
declining small enhancements, major enhancements, and new components. That is a mature
system deliberately closing the contribution door, not a failure.

## What actually goes in the system: components, recipes, snowflakes

Brad Frost's vocabulary, and the cleanest answer to "should this go in the design
system?" ([bradfrost.com](https://bradfrost.com/blog/post/design-system-components-recipes-and-snowflakes/)):

- **Design system component** - shared, content-agnostic, context-agnostic, built for
  maximal reuse. `Button`, `Card`, `Select`, `Table`.
- **Recipe** - a specific composition of design-system components, used consistently
  within one product but not agnostic enough for the shared system. `ProductCard`,
  `ContactCard`, `AddressField`. Often organisms in atomic-design terms.
- **Snowflake** - a genuine one-off needed to build a product, not reused elsewhere.
  Frost's example from an airline client: a `Seat` component for the seat-selection
  screen.

Two rules that follow from this:

1. **Things move down into the design system, not up out of it.** A recipe that proves
   widely needed can be promoted. Polluting the core library with product-specific
   components up front "gets noisy and messy" and has to be weeded out later.
2. **Recipes and snowflakes still deserve a home** (a separate "extras" library, or a
   `.storybook/recipes` directory) so they are visible and reusable within their
   product without contaminating the publishable core.

This directly answers the most common contribution request you will get, which is a
product team asking for their recipe to be adopted into the core system.

## Adoption: the actual hard problem

Sparkbox has surveyed design system teams annually since 2018. Adoption is persistently
near the top of the challenge list, and it correlates directly with perceived success.

From the [2021 Sparkbox Design Systems Survey](https://designsystemsurvey.sparkbox.com/2021):

| Top challenge | % of teams reporting it |
|---|---|
| Overcoming technical/creative debt | 47% |
| Contribution | 45% |
| Adoption | 44% |
| Staffing | 39% |
| Internal education | 36% |

The more telling cut is the correlation between reporting adoption as a challenge and
rating your own system's success:

| Self-rated system success | % who named adoption a top challenge |
|---|---|
| Very successful | 9% |
| Successful | 22% |
| Moderately successful | 49% |
| Slightly successful | 68% |
| Not successful | 83% |

Teams that consider their system unsuccessful are 9x more likely to name adoption as
their core problem than teams who consider it very successful. Also: 52% of agency
respondents said lack of adoption is the most common reason their clients' systems fail,
and among teams who had considered starting over from scratch, the top reason (12 of 25
responses) was adoption difficulty.

The [2022 survey](https://designsystemssurvey.sparkbox.com/) adds a second signal:
"parity between design and code" was reported as a top challenge by 37% of teams while
only 33% listed it as a priority, and staffing showed the largest gap of all between
being a challenge (35%) and being a priority (13%). Teams know they are understaffed and
are not prioritizing fixing it.

## How to measure adoption properly

Most systems measure "does this project import our library," which is nearly useless: a
page can import `Button` once and still be 90% hand-rolled markup.

**Razorpay's Blade solved this concretely, and this is the most exportable idea in this
entire skill.** Source: [engineering.razorpay.com](https://engineering.razorpay.com/cutting-deep-through-blade-23a72bcc3bcc).

The trigger was their co-founder asking a question every design system lead should
expect:

> "I don't know how the org is using Blade today. If you tell me that 4-5 projects are
> using the design system I don't know if that's a good number or not. How much of Blade
> are they actually using?"

Their answer, **% Page Coverage**:

```
% Page Coverage = Total Blade nodes / Total number of page nodes * 100
```

Implemented as a DOM walk over `document.querySelectorAll('body *')` that skips hidden
nodes, empty nodes, and media elements (`img`, `video`, `audio`, `source`, `picture`),
then counts nodes carrying `data-blade-component` or nested inside one. When they first
ran it, pages they believed were "built with Blade" scored only **40-50%**.

They then productized it three ways:
- Instrumentation events pushed periodically from every app into monitoring dashboards.
- A **Chrome extension** showing coverage for any page and visually highlighting the
  non-Blade nodes.
- A **[Figma plugin](https://www.figma.com/community/plugin/1258393250170675750/blade-coverage)**
  applying the same measure to design layers, attaching a coverage card to each frame,
  so a design can be checked for coverage *before handoff, before any code is written*.

And set org-wide OKRs directly on it:

> **O: All the frontend apps have a coverage of at least 20% of Blade components**
> - KR: New modules/apps have coverage > 70%
> - KR: 90% of traffic pages have at least 50% blade coverage

Note the deliberate asymmetry: new work is held to a high bar (70%), while migration of
old work targets only high-traffic pages, not a page count. That is a realistic
migration strategy, not a boil-the-ocean one.

Blade also runs **component and prop usage instrumentation** via
[react-scanner](https://www.npmjs.com/package/react-scanner), giving total occurrences
per component, usage split by project, and heavily-used vs unused props. They use it to
"demise any prop/token that wasn't used" and to decide which component families deserve
more investment. This is how you deprecate safely: with evidence, not guesses.

Dan Mall's [design system coverage](https://danmall.com/posts/design-system-coverage/)
analysis of the United Airlines homepage is cited by the Blade team as the inspiration.

## The rest of Blade's adoption operating model

Worth copying wholesale, because it is a first-party account of what actually worked at
a company serving 10 million businesses across ~13 products:

- **Blade Advocates** - product designers from each business unit who supply real
  use-cases during a component's design phase, review the outcome, and then evangelize
  it back to their own team. Solves the small-central-team reach problem without giving
  up quality control.
- **Component API decision records** - every non-trivial API is written up in a
  `_decisions/decisions.md` inside the component's own folder, with options and
  tradeoffs, then circulated to all frontend leads org-wide for sign-off before
  implementation. Slower per component, produces something teams trust.
- **Support channels**: open office hours twice a week for engineers and designers,
  labeled GitHub issues, ad-hoc JIRA support, a dedicated Slack channel.
- **Mandatory onboarding module** for every new designer before they touch a project, so
  the core team is not re-explaining fundamentals to each hire.
- **NPS surveys plus focus group discussions**, segmented by business unit and by stack
  (React / React Native / Svelte), covering flexibility-vs-restriction, design-to-code
  mapping, onboarding friction, and comparisons to alternative libraries.
- **Marketing the system internally.** They name "Platform teams don't take additional
  efforts to market the design system well" as one of six root causes of poor adoption,
  and treat release announcements as a real, recurring, deliberately catchy activity.
- **Branding and merchandise** for the system itself, plus rewards and recognition for
  adopters. Easy to dismiss as fluff; it is doing real work on voluntary adoption.

Their own six stated reasons adoption is hard, verbatim in substance:
1. Product teams chase business metrics and resist infrastructure overhead.
2. Developers are set in their workflows and resistant to change.
3. Designers feel design systems restrict creativity.
4. Leadership fears integration will affect timelines.
5. People are lazy; both roles avoid the friction of learning a new system.
6. Platform teams do not market the system enough.

## What a component documentation page must contain

Converged across every mature system reviewed (Carbon, Atlassian, Polaris, Spectrum,
Blade). No serious system ships less than this:

- Description and purpose (when to use, when *not* to use)
- Anatomy diagram with labeled parts
- Live interactive example
- Full prop/API table with types and defaults
- Every state shown explicitly (hover, focus, pressed, disabled, selected, error)
- Accessibility notes (keyboard interactions, ARIA roles used)
- Content/copy guidelines (how to word a modal title, a button label)
- Do's and Don'ts panels
- Ideally: the API decision record explaining *why* the API is shaped that way

## Health signals: is the system healthy or just busy?

Curtis's diagnostic, which is about cadence rather than artefact count:

For an established system:
- How regularly is it releasing fixes and small enhancements aligned with real demand?
- How often is it delivering larger capabilities that make an impact?

For a system getting started:
- How quickly is it delivering and validating value incrementally?
- How much is it sharing with, and getting feedback from, pilots and early adopters?

A team producing lots of tokens, docs, and process while shipping nothing adopters
notice is busy, not healthy.

## Token and component change management

- Treat a semantic token or component prop as a public API the moment anything consumes
  it. Renaming or removing one is a breaking change requiring a deprecation window and
  ideally a codemod, not a silent rename.
- Additive changes are safe. Changing an existing token's *meaning* is the dangerous
  one, because nothing errors; it just silently looks wrong somewhere.
- Keep the old token as an alias to the new one for at least one release, and instrument
  who still references it before removing.
- Primitive-layer value changes are low risk *only if* every consumer goes through the
  semantic layer. That is the entire argument for the semantic tier existing.
- Version tokens alongside the component library. A token package a major version behind
  the component package is a very common source of real visual drift.
- Component lifecycle stages worth publishing explicitly: **Core** (widely used, actively
  maintained), **In Review** (being evaluated), **Deprecated** (still working, being
  phased out), **Archived** (unsupported). Without published stages, nothing ever gets
  removed and the library only grows.

## Why design systems die

Cross-checked across Sparkbox survey data, first-party postmortems, and practitioner
accounts:

1. **Building everything at once**, so nothing ships early enough to prove value.
2. **No adoption instrumentation**, so the team cannot prove worth, cannot prioritize,
   and deprecation is guesswork.
3. **Federated-only staffing** - deallocating central people and expecting a community
   to carry it. Curtis: "you deallocated people responsible for doing the work. What did
   you expect?"
4. **Absorbing product-specific components** because one team asked loudly, which
   complicates support and irritates every other team.
5. **No internal marketing**, so teams simply never learn a component exists.
6. **No support channel**, so early friction becomes "I'll just build my own."
7. **Design and code drifting apart.** Curtis, asked what breaks first at scale:
   "Probably the divergence of code and design assets and docs across disciplines."
8. **Treating it as a one-time deliverable** with no roadmap, support model, or metric.
9. **Burning out the founding team** on a months-long build with no interim shippable
   value.

## Contested

- **Centralized vs federated.** Settled more than most people realize, given Curtis's
  own retraction: central capacity is non-optional, federated contribution is an
  optional addition on top. The genuine open question is only *how much* federated
  participation is worth the curation cost.
- **Whether contribution is worth pursuing at all.** Curtis is openly skeptical
  ("Frankly, I hate the word 'contributions'"), preferring to focus on where demand for
  reuse actually emerges. Atlassian restricts contributions to fixes. Other mature teams
  invest heavily in contribution programs. Reasonable people differ.
- **Buying a docs platform (zeroheight, Supernova) vs rolling your own.** Below roughly
  20 people it is generally not worth the spend; Storybook plus MDX covers it.
