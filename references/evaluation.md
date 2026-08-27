# Evaluation: how to test whether the system works

A design system used by agents needs the same thing any other production dependency
needs: a way to tell whether it is working, before it is too late to change. Agents
generate more UI per day than any human can review by eye, so "we will notice if it
drifts" stops being true.

There are four levels. They answer different questions and none substitutes for another.

| Level | Question | Method | When |
|---|---|---|---|
| L0 | Is the system internally well-formed? | Static checks on the system itself | Every commit |
| L1 | Does an agent using it produce compliant output? | Generation eval on held-out tasks | Before rollout, then per release |
| L2 | Is the system **sufficient**? | Build a real screen, compare side by side | Before rollout, then quarterly |
| L3 | Is it actually adopted? | Coverage and usage instrumentation in production | Continuously |

The most common mistake is running L0 and L3 and believing you have covered L1 and L2.

---

## L0: is the system well-formed

Automated, deterministic, runs in CI. None of these need an LLM.

| Check | Passes when |
|---|---|
| Token resolution | Every semantic token resolves to a real primitive; no dangling references |
| No literals | No hex, `rgb()`, `hsl()`, or off-scale spacing anywhere in component source |
| Tier discipline | Components reference semantic tokens only, never primitives |
| Registry integrity | Every component is declared once, names unique, every export resolves |
| Contrast | Every foreground/background token pairing the system can actually produce clears its bar, **in every theme** |
| Interaction states | Every interactive component declares hover, pressed, focus, disabled |
| ARIA mapping | Every interactive component maps to a named APG pattern or documents why not |
| Elevation ordering | In each theme, elevation surfaces form a strictly monotonic sequence |

Two rules about these gates that matter more than the list:

**Test contrast on token pairings, not on rendered components.** Contrast is a property
of a token pair. Checking rendered screens finds only the combinations you happened to
build; checking pairings finds the ones you will build next month.

**Any gate that has never failed should be assumed broken.** A lint rule that passes on
its first run is indistinguishable from a no-op. Give every rule a fixture test that
deliberately violates it and asserts the rule fires, plus one fixture asserting the clean
baseline still passes so the rule cannot go vacuous over time.

---

## L1: does an agent produce compliant output

This is the level almost nobody runs, and it is the one that tells you whether the system
survives contact with generation.

### The protocol

**1. Fix the rubric and the pass condition in writing before you run anything.**
Not after. The entire value is that it cannot be relitigated when a prettier but
non-compliant output is sitting in front of you.

The pass condition that matters most, stated explicitly:

> A slightly wrong-looking screen built entirely from existing components beats a
> pixel-perfect screen containing bespoke ones. Inventing a component to match the
> reference more closely is a failure, even if it looks better.

Without this written down in advance, every eval degrades into "it looks good enough."

**2. Use a reference from a different product.** If you test against the product the
system was derived from, correct component mapping is indistinguishable from source
recognition. Take a screenshot from a comparable product the system has never seen.

**3. Vary surface *type*, not screen count.** Five transactional screens will all miss
the same gap. Cover: a dense list or grid, a form, an empty or error state, a marketing
or promotional surface, and a settings or detail view. Five different *kinds* beats
twenty of the same kind.

**4. Give the agent only what a real user would have.** The skill or rules file, the
registry or MCP, and the reference. No hints, no project context, no access to the eval
rubric.

**5. Score objectively, from the artifact.** Never accept the agent's self-report. A
model will truthfully say "no deviations I felt were significant" while having hand-built
two visual treatments out of raw elements.

### The metrics

| Metric | Target | Why |
|---|---|---|
| Component reuse rate | >90% of elements are registry components | Measures discipline |
| Invented components | **0** | The failure that passes every other gate |
| Raw literal values | **0** | Hex, off-scale spacing, arbitrary values |
| Raw structural markup | Near 0 | See below, this is the one people miss |
| Theme rendering | Clean in every theme | Hardcoded values surface as theme bugs |
| Accessibility | Automated checks pass | Agents under-produce accessible markup by default |
| Determinism | Same prompt twice, no material difference | Non-determinism means the constraints are too loose |

**The metric almost everyone gets wrong.** A component-reuse percentage counts only
component elements. A visual treatment hand-built from raw `<div>`s and `<span>`s scores
100% because those are not components. Add a second metric that counts **raw markup
carrying visual classes** (`bg-`, `border-`, `rounded-`, `shadow-`, `ring-`), split into:

- **Structural** (building a visual treatment by hand) - this is the real failure
- **Typography only** (applying a text token to a span) - this is correct usage

That split is what surfaces genuinely missing components. A 100% reuse score with three
hand-built structural treatments means your system is missing three components.

### Published harnesses worth copying

You do not have to invent the harness shape. Two real ones exist:

**Figma's Code Connect eval** ([figma.com](https://www.figma.com/blog/the-benefits-of-code-connect-in-mcp/))
is the most complete published example: 27 test cases, two design systems at different
coverage levels, two models, LLM judges scoring against known-good reference snippets on
a 1-4 Likert scale across correctness, code quality, maintainability, completeness, and
best practices. They ran variants multiple times to discount outliers; some runs took
over 24 hours end to end.

Their scale is directly reusable:

| Score | Meaning |
|---|---|
| 1 | Broken: syntax errors, crashes, or completely fails the task |
| 2 | Poor: partially works, bad patterns or major correctness problems |
| 3 | Good: solid, minor issues only |
| 4 | Excellent: clean, idiomatic, correct, follows best practices |

Their headline finding is also the most useful thing to know before running your own:
**the single biggest factor moving the numbers was coverage**, meaning how much of the
design is composed of system components that have a real code mapping. Output quality is
capped by mapping coverage, not by model choice.

**Spotify** built a custom MCP evaluation framework benchmarking generated components
against multiple LLMs, both in code and visually, on the stated reasoning that "we can't
just launch and hope for the best" ([Into Design Systems](https://www.intodesignsystems.com/blog/design-system-not-ready-for-ai-agents)).

### Validate the instrument before trusting the number

Scoring scripts have bugs that produce confidently wrong conclusions. Two real classes:

- A regex counting components as `/<([A-Z][A-Za-z0-9_]*)/g` also matches TypeScript
  generics, registering `Record` in `useState<Record<string, number>>` as an invented
  component. Require a JSX terminator: `/<([A-Z][A-Za-z0-9_]*)(?=[\s/>])/g`.
- A coverage sweep over `document.querySelectorAll('body *')` counts user-agent defaults
  as drift: black on wrappers that never paint text, default link blue on anchors whose
  visible text is colored by an inner span. Count an element only if it paints its own
  text (a direct non-empty text node) or its own background.

Before acting on any adoption or compliance score, run the scorer on a case where you
already know the right answer.

---

## L2: is the system sufficient

**This is the level automated metrics structurally cannot reach, and the most important
one.**

Gates prevent drift. They cannot tell you the system is incomplete. A screen can score
100% on every automated check and still be wrong, because every check asks "did you use
the system correctly," never "was the system enough."

### The method

Rebuild a real screen from a real product using only the system, then put the two images
side by side and enumerate every difference. That is the whole method. It is manual and
it is not optional.

Categorize what you find:

1. **Missing components** - nothing in the system expresses this
2. **Components too thin** - the right component exists but cannot carry the real content
   (a card exposing one metadata line where the real page needs three)
3. **Internal inconsistencies** - the system contradicting itself, usually invisible until
   you see several components together

Category 3 is the highest-value find and the one you will never get from a component
gallery, because it only appears when multiple components sit on the same screen.

### Two failure classes only composition reveals

**Slot collisions.** Two optional slots each render correctly alone and collide when both
are filled at once. A gallery with default props never exercises this. As component APIs
move toward slots and composition, this class gets more common.

**Semantic misuse.** An agent reusing a component that is structurally close but
semantically wrong (a retailer tile used as a category tile) scores as a *success* on
every metric, because a real registry component was used. No automated gate currently
detects this. Only a human reading the output catches it.

The output of L2 is not a score. It is an evidence-backed inventory of what the system
lacks, derived from building with it rather than reviewing it.

---

## L3: is it adopted in production

Import counts are nearly useless. A page can import `Button` once and be 90% hand-rolled.

**Coverage is the metric.** Razorpay's Blade formula, the clearest published version:

```
% Page Coverage = system nodes / total meaningful page nodes * 100
```

Implemented as a DOM walk skipping hidden nodes, empty nodes, and media elements, then
counting nodes tagged as system components or nested inside one. When Razorpay first ran
it, pages the team believed were built with the system scored **40-50%**.

Productize it in three places:
- Instrumentation events from every app into a dashboard
- A browser extension showing coverage and highlighting non-system nodes visually
- A plugin applying the same measure to design files **before handoff**

Then set goals on it. Their published OKR shape, with the asymmetry worth copying:

> All frontend apps have at least 20% coverage
> - New modules and apps have coverage > 70%
> - 90% of traffic pages have at least 50% coverage

New work is held to a high bar; migration targets high-traffic pages rather than a page
count. That is a realistic migration strategy.

Pair coverage with **component and prop usage instrumentation** (`react-scanner` or
equivalent) so deprecation is evidence-based rather than guesswork, and with periodic NPS
plus focus groups segmented by team and stack to catch friction a usage number cannot see.

---

## Accessibility deserves its own gate

Agents do not self-correct on accessibility. One 2026 study measuring 21,880 assessments
across five WCAG criteria found AI-generated interfaces achieved **29.0% compliance
overall**, with color contrast at 26.8% and correct use of color at 19.2%
([ACM](https://dl.acm.org/doi/full/10.1145/3800424.3800430)). Treat the exact figures as
one study, but the direction as settled.

Consequence: automated accessibility checks (axe-core, `eslint-plugin-jsx-a11y`, a
Storybook a11y addon) belong in the same CI loop the agent already treats as
authoritative, and token design should make invalid pairings unexpressible rather than
relying on the agent to check contrast.

---

## Common evaluation mistakes

1. **Writing the rubric after seeing the output.** The rubric exists to constrain you at
   the moment you are tempted.
2. **Accepting the agent's self-assessment.** Score from the artifact.
3. **Testing against the source product.** Recognition masquerades as correct mapping.
4. **More screens of the same kind.** Vary the surface type instead.
5. **Believing 100% means done.** It measures discipline, not sufficiency. Run L2.
6. **Trusting a gate that has never fired.** Fixture-test it.
7. **Trusting a scorer you have not validated.** Run it on a known answer first.
8. **Skipping the side-by-side because CI is green.** CI cannot see missing components.

---

## Minimum viable eval

If there is budget for only one thing, do this:

1. Write the pass condition down before you start.
2. Take one screenshot from a comparable product the system has never seen.
3. Give a fresh agent only the skill file and the registry, and ask for the screen.
4. Score it yourself against the rubric: reuse rate, invented components, raw literals,
   raw structural markup, theme rendering.
5. Put the result side by side with the reference and list every difference.

That is roughly an hour and it will tell you more about your design system than any
amount of documentation review.
