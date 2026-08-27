# Designing for 2026+: when the primary consumer is an AI agent

Everything else in this skill (tokens, color, naming, components, governance) still
holds. What changes in the agent era is the **delivery mechanism** and **what counts as
enforcement**. This file is about that layer. Read it alongside, not instead of, the
rest of the skill.

## First, the honest answer on Figma

The premise "Figma might not even be relevant" deserves scrutiny, not agreement. Two
survey data points, not vendor announcements, are the right place to start:

- The [UX Tools State of Prototyping survey](https://survey.uxtools.co/spring-2026)
  (22,000+ designers surveyed since 2017) still finds **Figma the most-used design tool
  in 2026.** What is changing is how it gets used, not whether it gets used.
- The [State of AI Design Report 2026](https://stateofaidesign.com/chapters/tools):
  frequent AI usage among designers jumped from **54% to 91% in one year**; **50% of
  surveyed designers have shipped code to production**; **76% have used an AI coding
  tool** (Claude Code, Codex, Cursor, Copilot) and **85% have used one of those and/or an
  app builder** (Lovable, Replit, Bolt). Adoption of internally-built AI tooling scales
  sharply with company size: **74% at 2,000+ employee companies, 48% at 501-2,000, 34%
  at 51-500, 26% at 1-50.**

Read that last stat carefully: code-fluency in design concentrates at large,
well-resourced companies, not evenly across the industry. A 10k+ user company is exactly
the size where this is now normal practice, not an outlier bet. A five-person startup is
a different story.

**A specific bad-evidence pattern worth naming:** in April 2026, Anthropic shipped Claude
Design and Figma's stock dropped about 7% in a day, producing a wave of "Figma is dying"
takes. A one-day stock move reacting to a new competitor's launch is not evidence of
obsolescence; treat it as noise. The actual data points the other way. Figma's own
**State of the Designer 2026** survey (906 designers, run with NewtonX) and its separate
**2026 AI Report** (8,403 responses over three years, 639 qualitative interviews) found
that as AI made code generation more accessible, design expertise became *more* valuable,
not less: **90% of respondents building AI-powered products said design is at least as
important as before AI, and 57% said it is more important**
([figma.com/blog/2026-ai-report](https://www.figma.com/blog/2026-ai-report/)).

**What Figma actually shipped at Config 2026** (June 2026) confirms this: not retreat,
absorption. In Figma's own words: "For years, the design industry has talked about
'design versus code'... but this is a false debate. Design is a process. Code is
material, just like images, vectors and design layers... So we are introducing code
layers in Figma."
([figma.com/blog/config-2026-recap](https://www.figma.com/blog/config-2026-recap/)).

- **Code layers** (closed beta): a design layer that is literally runnable code on the
  canvas, convertible back and forth between design layer and code layer with a single
  click. Clone a repo, extract interaction flows from existing code as inspectable
  design layers.
- **Figma Make now edits your real production codebase**
  ([figma.com/blog/figma-make-now-on-your-local-code](https://www.figma.com/blog/figma-make-now-on-your-local-code/)),
  not just prototypes: clone a GitHub repo, select an element, the agent finds the
  relevant code and edits it so the UI reflects what you designed, then opens a real
  pull request with branches and revertible commits.
- **The Figma MCP server** (remote) lets agents build and update frames, components, and
  variables *inside* Figma using your design system as source of truth, bidirectionally.
- Dylan Field's own framing, stated on stage: "Code is not the opposite of design. Code
  is material for design."
  ([qubika.com Config recap](https://qubika.com/blog/figma-config-2026-announcements-for-designers/))

Figma's strategy is not "stay a design-only tool." It is "become the place where design
and code are the same material." That is a materially different claim than "Figma is
becoming irrelevant."

**What genuinely supports the other side of the premise:** Nathan Curtis, the single
most credentialed voice in this field (25+ years, 80+ systems consulted), writing in his
own newsletter in early 2026:

> "I suspect I'll be broadening how to extract, transform and inject that data using
> things like Figma's Rest API and MCP as our field shifts, relates and **may even move
> away from our Figma canvas** as generative approaches take us somewhere else."
> ([Design Systems Collective](https://www.designsystemscollective.com/were-focused-too-much-on-design-tokens-nathan-curtis-on-design-systems-today-a329fdd79d4c))

And, on what actually consumed his team's time in 2025: "I certainly did NOT predict
that my team's 2025 would be spent recording architectural decisions about components by
reviewing copious examples formatted in yaml and markdown instead of Figma and
Storybook. But here we are."

A real, lived account of the split this creates: a first-hand 2026 report describes a
designer who "started skipping Figma altogether, going straight to Claude Code, not as a
statement, just as the faster path," on a team that had merged 350+ PRs from designers
using Cursor and Figma MCP by Q4
([uxdesign.cc, "The Design Engineer Symptom"](https://uxdesign.cc/the-design-engineer-symptom-what-a-rising-job-title-reveals-850d5e4fd9cc)).
Both things are true at once: some designers go straight to code for speed, and Figma
remains what most teams still use for exploration, review, and non-technical stakeholder
buy-in. **Design as source of truth without any visual medium is currently a minority
workflow, not the emerging default.** Build for both; do not bet the whole system on
either disappearing. What is actually dying is the old sequential handoff (sketch, spec
document, wait for an engineer to translate it by hand), not the canvas itself.

**The calibrated verdict:** Figma is not dying and is not being displaced by a single
challenger tool. But the *center of gravity for design decisions* is genuinely moving
toward structured data and code, and Figma itself is trying to become one visual surface
onto that data rather than the sole source of truth. Treat Figma as a surface, not as the
database. Design the system so code (or a platform-neutral spec, see below) is the
source of truth and Figma is a view onto it, not the reverse. This is the same
conclusion the rest of this skill already reaches for unrelated reasons (code as source
of truth, synced to Figma), but the 2026 evidence makes it a stronger default, not a
hedge.

**Design engineer is a real, rising role**, not a buzzword: "the most effective digital
products... are increasingly shaped by a new kind of operator: the design engineer"
([Peerlist, April 2026](https://peerlist.io/shuvrojit/articles/the-rise-of-the-design-engineer-in-2026)).
Qubika's read on Config 2026 is the sharpest single line on what this means for
individual designers: "The designer who understands how a component library works, who
can reason about interaction states and edge cases, who thinks in systems rather than
screens: that designer gains significantly more leverage with Code Layers than someone
who has operated exclusively in the visual layer." Code literacy is becoming table
stakes for design leverage, not a separate career track.

## Why raw Figma data is the wrong thing to feed an agent, even via MCP

A second, independent Curtis finding, and arguably the more actionable one. From
[Components as Data](https://medium.com/@nathanacurtis/components-as-data-2be178777f21):
"However you get it, MCP, REST API, plugin, raw Figma data has limitations
when thrown to AI to generate components at scale."

Four concrete reasons he gives: it is **expensive and ephemeral** (MCP reprocesses a
large payload every call, nothing persists); it is **a noisy dumping ground** (his own
number: "26,901 properties when you need roughly 350", 45 full
variant copies when what matters is 15 subtle differences between them); it is **not
clean** (even mature libraries have inconsistent naming, orphaned variants, dead
variables); and **Figma favors Figma** (its property names are Figma's own vocabulary,
not your platform's: `paddingLeft` vs iOS's `.padding(.leading)` vs CSS's
`padding-inline-start`).

His answer is not "stop using MCP." It is **insert a compilation step**: extract
Figma's actual state once, deterministically, into a spec that is complete, compact,
precise, translated to platform-neutral names, versioned, and human-readable. He
reports fanning out "a TS contract, state-aware CSS, a React scaffold, and Storybook
stories for 50 components" from generated specs in one pass. **The general principle,
independent of tooling choice: live MCP-fetching a design file at generation time is
fine for one-off inspection. It is the wrong pattern for repeated, at-scale code
generation. Compile a structured spec once, version it, have agents consume the
compiled spec.**

## The controlled evidence: Figma and Coinbase measured this, not just claimed it

Most of this space runs on unverified vendor claims. This is a rare exception, and it is
worth taking seriously precisely because it is a real eval harness, not a testimonial:
[Figma ran a controlled comparison and published the numbers](https://www.figma.com/blog/the-benefits-of-code-connect-in-mcp/).

Coinbase's Design Systems team (serving its largest consumer products) moved to
agent-driven development and invested in Code Connect specifically so agents build with
real components instead of reinventing them. Frontend engineer Erich Kuerschner ran the
same design, prompt, and model with and without it: "Without Code Connect, the agent
would sometimes fabricate its own version of components: say, build a stepper out of
progress bars."

Figma then built a proper eval harness: 27 test cases, two design systems (Figma's own
Simple Design System at ~20% Code Connect coverage, and Figma's larger internal Pattern
Library at ~6% coverage), two models (Claude Sonnet 4.5 and Opus 4.7), LLM judges scoring
correctness, code quality, maintainability, completeness, and best practices on a 1-4
Likert scale. Median results with Code Connect versus without:

| Metric | Change with Code Connect |
|---|---|
| Code quality (1-4 Likert scale) | +1 (2 to 3) |
| Token usage | -29.5% |
| Task duration | -19.6% |

What that hides in the median: without Code Connect, the agent hand-built tabs and
buttons as raw divs and CSS that *looked* right but were not even functional (a
hardcoded "selected" tab state, no real interactivity). With Code Connect it imported
the real `Tabs` and `Button` components and produced working code, finishing in about
77% of the time and 62% of the tokens on that specific case. Figma's own conclusion:
"The result might look correct visually, but the code might not be what you'd want
committed to your codebase. To do a good job, agents need more explicit, structured
context, in this case, a real code snippet for how to implement the component in your
codebase."

**The variable that moved these numbers most was coverage**: how much of the handoff
design is actually composed of design-system components with a real code mapping, and
how much of that mapping is actually Code Connected. This is Blade's "% page coverage"
metric (`governance.md`) applied one layer earlier: an agent's output
quality is capped by how much of your system is actually mapped, not by which model you
use.

## The architecture that actually works: foundations always-on, components on-demand

This is the single most important correction to make versus a naive "just wire up an
MCP server" approach, and it comes from a first-party failure report, not theory.

Diana Wolosin, building Indeed's design-system MCP for a design system serving **more
than 2,000 R&D designers and engineers**
([Design Systems Collective, May 2026](https://www.designsystemscollective.com/fully-machine-readable-design-systems-3d43329ec3e3)):

> "A design system MCP that returns the Card, Button, and props on demand, but generates
> UI that still lands with broken typography hierarchy, inconsistent spacing, incorrect
> icon usage, and layouts that felt structurally unlike Indeed... I assumed that
> exposing our design system to LLMs through an MCP would solve compliance, and it does,
> partially. It solves component retrieval. **It doesn't solve for quality.**"

The diagnosis: "An MCP is on-demand. It returns only what the prompt asks for... A
prompt for 'build me a card' returned Card and Button knowledge. It did not return the
spacing token grammar that determines whether the card breathes correctly inside a
page... Components are on-demand: the LLM fetches them when the prompt asks. Foundations
cannot be fetched. They have to be attached to the file before the LLM writes a line of
UI code. Different timing, different mechanisms."

Her team's fix, called **progressive context disclosure**: package foundations,
quality/taste guidance, implementation patterns, composition recipes, and spatial rhythm
as separate layers in a plugin (the packaging convention both Anthropic's Claude Code and
Cursor use to deliver AI context), each loaded at the moment the model can actually act
on it, not dumped into one prompt up front. A single skill file indexes and routes to the
right layer for the task, then calls the design-system MCP for the specific component.
The rule this produces, matching the pattern independently confirmed by the [Into Design
Systems "Agentic Design Systems" guide](https://www.intodesignsystems.com/agentic-design-systems)
(August 2026):

1. **Always-on foundations**: spacing grammar, typography hierarchy, icon conventions,
   brand compositional intuition. Loaded whenever a relevant file is open, before the
   model writes a line.
2. **MCP on-demand for components**: specific component API, props, and variants,
   fetched when the prompt actually needs one.
3. **A thin orchestration layer** (a skill index, or `AGENTS.md`/`CLAUDE.md`) that routes
   to both rather than repeating either.

Her team grounded the foundations layer in real evidence rather than one designer's
opinion: a Sourcegraph MCP audit across **14 production codebases, 1,697 files, and
6,147 spacing-token occurrences**, which surfaced exactly six spacing tokens and four
global recipes that recur on every product surface. That became the baseline for the
spatial-rhythm layer. Calibration runs against real prompts also caught a documentation
bug: cards kept generating with a deprecated left-border accent because the docs still
prescribed a style the team had abandoned years earlier. The model was not wrong; the
documentation was. Fixing the doc fixed the output on the next run.

Her sharpest framing, worth carrying forward as this whole area matures: **"The frontier
of our design work is no longer documentation. It is context architecture."** And a
distinction worth keeping: some of what a design system encodes is correctness (a
component used right, spacing that follows the rules), checkable mechanically; some of
it is judgment (does the layout breathe, does it feel like this specific product), which
has to be reduced to something specific enough for a model before it can be encoded at
all, and cannot simply be automated away.

Spotify independently arrived at a similar split, rebuilt as three layers: foundation
(tokens/primitives), style (visual appearance), behavior (interaction logic) - and
built a custom MCP evaluation framework that benchmarks generated components against
multiple LLMs, both in code and visually, because "we can't just launch and
hope for the best" ([Into Design Systems](https://www.intodesignsystems.com/blog/design-system-not-ready-for-ai-agents)).

**Do not ship a single `DESIGN.md` and assume it scales.** Wolosin names this
explicitly as the failure mode of the naive approach used by tools like Stitch and
Lovable: works for a small palette and a handful of components, collapses once "design"
means density, rhythm, composition, and brand expression at real scale.

## JSON for the machine contract, Markdown for the narrative, and now a real number

This was already a rule of thumb; it now has a benchmark behind it. From the same Into
Design Systems piece, describing a real test (77 components parsed from MDX docs, 8 MCP
configurations, 1,056 prompts):

| Format | Tokens per query | Coverage | Failure mode |
|---|---|---|---|
| Markdown docs dumped into MCP | ~30,000 | 82% | Hallucinations present |
| Structured JSON | 80% fewer tokens than Markdown | Higher accuracy | - |

Cost consequence cited: roughly $300/year vs $1,500/year at the same query volume. The
rule this supports: **component APIs, props, and variants are a contract; they belong
in JSON.** Narrative guidance, rationale, and do/don't judgment belong in Markdown for
the model to read as prose, not as a schema it has to parse to find one field.

Diana Wolosin's own framing of the same point: "Our docs are written for
humans. The new user, AI, needs structured metadata, not documentation prose."

## Component specs as platform-agnostic structured data (Nathan Curtis, 2026)

Curtis's current work is the clearest instance of "the design system as data" done at
production maturity, and it directly targets the same problem this file opened with:
Figma is expensive, ephemeral, and incomplete as a spec.

From his April 2026 Substack piece, ["Figma Component Specs on Command"](https://nathanacurtis.substack.com/p/figma-component-specs-on-command):

> "That moment in between - what we've called 'handoff' that extracts a Button's
> anatomy from 45 variants, diffs states, resolves token references - has no ambiguity
> at all. It's mechanical, known, deterministic. **The answers are in the data.**"

His own description of the failure mode this fixes is worth quoting for how concrete it
is: "When I watch an AI agent spend four minutes crawling a Figma file to answer a
question I already knew the answer to, I get impatient... The agent was reading 45
variants, one by one, not even trying to infer which properties change across states.
Isn't even an inference problem! It should be a diff!" His conclusion, the single most
important operating principle in this file: **"AI should be reading, not writing, these
specs from Figma."**

On why Figma alone cannot be the spec: "Figma lacks a native model for ARIA
roles and screen reader labels, but your components ship with each. A cross-platform
spec needs to be neutral ground, not one tool's more limited view."

His tool, **Specs** (formerly Anova, now "Specs 2"), ships both as a Figma plugin
([specsplugin.com](https://www.specsplugin.com/)) and a versioned CLI
([github.com/DirectedEdges/specs](https://github.com/DirectedEdges/specs)); it is a
real, shipping product, not a concept. It extracts, per component: **anatomy** (every
element by type), **props** (with every per-variant difference detected), **styling**
(token references, named styles), **layout** (hierarchy, flex vs absolute),
**bindings** (what drives content/instances/conditionals), **subcomponents**, and
**slots**. Output is schema-valid YAML/Markdown, deterministic and versioned. The
compression numbers are the whole argument for it:

| | Raw Figma data | Generated spec |
|---|---|---|
| Button, 45 variants | 1.38 MB, 42,000+ lines, 280 nodes, 26,000+ properties | 10 KB, 442 lines, 4 anatomy elements, 15 real variant rules |
| Compression | | 99.25% smaller, 134:1 |
| Action List with subcomponents | 2.6 MB | 14 KB, 99.47% reduction, 188:1 |
| Time per component | | ~1 second (CLI) to ~10 seconds (plugin) |
| AI token cost to generate | 25,000-100,000+ tokens if an agent infers it instead | Zero - a deterministic script, not an LLM call |

His stated design principle, worth adopting directly: **"computation over inference."**
Where an answer can be extracted mechanically and deterministically (a Button's actual
anatomy from its 45 real variants), do that, and reserve agentic/LLM inference only for
what genuinely requires judgment. Curtis's own comparison table:

| | Computation | Inference |
|---|---|---|
| Repeatability | Deterministic, mechanical | Errors, gaps, overconfidence |
| Output | Schema-valid YAML/MD, testable, versionable | Unstructured Markdown, unpredictable |
| Platforms | Schema-based, cross-platform | Guessing intent per platform |

This is the concrete instantiation of Curtis's broader 2026 claim (see
`components.md`) that component architecture, not tokens, is the field's real
unsolved problem, applied specifically to making that architecture legible to machines.

**Practical takeaway even without adopting his tooling:** treat "what is this
component's real anatomy, given every variant that actually exists" as a question with
a mechanically correct answer, not a documentation-writing exercise. If your component
library has drifted (a variant nobody remembers exists, a state that's visually
identical to another), a deterministic extraction tool will surface that drift; a
human writing docs from memory will not.

**Examples, not just props, are the other half of the same principle.** A companion
piece, [Component Examples as Data](https://nathanacurtis.substack.com/p/component-examples-as-data)
(May 2026), extends the logic to usage examples, and connects directly to the
props-versus-slots shift already covered in `components.md`: "The craft that
makes a component quality clear to a designer is the same that makes it legible to a
machine. Props aren't enough." As components move toward composition and slots, a flat
prop table stops being sufficient documentation, because the actual valid shapes of a
component are compositions, not configurations. Curtis now ships `examples` as a first
class spec output alongside `api` and `variants`, with content expressed as structured
placeholders (`{Title}`, `{Description}`) that carry enough intent to be swapped rather
than literal copy. **Practical takeaway: component documentation needs a set of real,
ready-made composition examples, not just an API table, because that is what both a new
designer and an agent actually copy from.**

## The Tailwind-prior problem, and the answer that is actually emerging

Models are heavily trained on default Tailwind. If your system renames or removes the
default scale, an agent will still reach for `p-4`, `bg-red-500`, `rounded-lg` from
sheer training-data gravity, whether or not those classes resolve to anything in your
project.

The 2026 industry answer is not "fight the prior harder with more rules." It is
**stop fighting it: align your token naming to the ecosystem convention the model
already knows.** A concrete real-world case, `force-ui`
([github.com/lobiklukas/shadcn-findings](https://github.com/lobiklukas/shadcn-findings)),
rebuilding its v2 architecture specifically for this reason:

> "**Ecosystem over isolation** - Standard tokens mean instant compatibility with
> shadcn, v0, community registries, and AI tools... **AI-native from day one** - the
> design system ships AI-consumable knowledge (skills, rules, guidelines) alongside
> components, leveraging existing ecosystem MCP tools."

Concretely: v1 had ~500 custom CSS variables in a Fluent-UI-inspired scheme
(`neutral-background-1-rest`, `brand-foreground-2-hover`). v2 builds on shadcn's default
token names (`--primary`, `--secondary`, `--muted`) and layers custom identity on top as
a *style*, not a competing naming scheme. This is a direct, evidenced update to the
naming guidance elsewhere in this skill: **for a from-scratch system in 2026, aligning
primitive-tier naming to Tailwind/shadcn conventions is a legitimate default reason to
choose descriptive-primitive names over a fully custom scheme**, specifically because it
reduces the model's fight against its own training prior. This does not change the
semantic-tier guidance (still name by purpose, not by value); it changes the calculus at
the primitive tier specifically.

## The reliability data behind "rules lose to code"

Before the ranked list, the actual numbers behind why prose loses, from a January 2026
internal research document by a Meta team building a component system specifically for
AI-generated code (`facebook/astryx`, explicitly framed as an exploration, not a finished
product):

| Strategy | Enforcement level | Reliability |
|---|---|---|
| Context docs / rules files | None, a suggestion | Low |
| System prompt instructions | None | Low |
| JSDoc / inline comments | None, competes with surrounding code as a signal | Low |
| Schema-enforced structured output | API-level | High, but only for JSON, not JSX/TSX |
| Typed component APIs | Compile-time | High, but reactive not preventive |

Their stated reason context injection fails: **"Context injection fails because it's
optional. The AI can always ignore guidance."** Two further findings from the same
document worth carrying:

- **JSDoc and inline comments do not work.** "Not reliably read... Comments are one
  signal among many (surrounding code, open tabs, cursor context)" with "no privileged
  status." Their conclusion, stated as a resolved open question: **types beat comments,
  because types are enforced and docs are ignored.** If you are tempted to document a
  constraint in a comment, encode it in the type instead.
- **The strongest single number in favor of typed APIs:** type-constrained code
  generation **reduces LLM code generation errors by more than 50%**
  ([arxiv.org/abs/2504.09246](https://arxiv.org/abs/2504.09246)). This is the clearest
  published justification for spending design-system effort on making invalid states
  unrepresentable in the type system rather than on writing more guidance.

Also from the same source, on the failure mode this whole file exists to prevent:
AI "optimizes for local correctness, not systemic soundness," generating arbitrary
values like `mt-[13px]` instead of tokens, with "different prompts produc[ing] different
class combinations for same intent." That is design-system drift stated precisely.

On types specifically, grounded in real code-generation research rather than assumption:
LLMs pass all tests on the **first attempt only 46-65%** of the time
([arxiv.org/html/2412.14841v1](https://arxiv.org/html/2412.14841v1/)); models given an
error-feedback loop ("Reflexion") recover to **95-98%**. The reframe this demands, used
again below under the ranked enforcement list: **a type system is not there to make the
agent get it right first try. It is there to turn a silent bug into an instant compile
error the agent's own tool loop repairs automatically, often within the same turn.**
That is a different, more honest claim than "types prevent mistakes." (Their specific
finding on `llms.txt` gets its own full section further down.)

## Enforcement, ranked strongest to weakest

Consolidating both research passes into one ranked list. Prose instructions are the
weakest gate and should never be the only one:

1. **Making the violation impossible to compile.** Tailwind v4's `@theme` with
   `--spacing: initial` means an off-scale utility like `p-5` generates no CSS at all.
   Stronger than lint because there is no way to accidentally ship it; it does not exist.
2. **Lint rules that fail CI.** `no-restricted-imports` for deprecated/non-registry
   components as a hard error. Real, shipping examples:
   [`@metamask/eslint-plugin-design-tokens`](https://github.com/MetaMask/eslint-plugin-design-tokens)
   and Atlassian's own [`eslint-plugin-design-system`](https://atlassian.design/components/eslint-plugin-design-system/ensure-design-token-usage)
   ("Ensure design token usage") both ship in production today.
3. **Strict types as a contract**, detailed below with the discriminated-union example.
4. **A registry allowlist enforced by import resolution**, so importing anything not in
   `registry.json` fails the build.
5. **Visual regression as the last-resort net**: Chromatic, Percy, Playwright's
   `toHaveScreenshot()`. Necessary because an agent can generate more screens per day
   than a human can eyeball.
6. **Accessibility checks in CI** (axe-core, eslint-plugin-jsx-a11y, Storybook's a11y
   addon), specifically because agents reliably under-produce accessible markup unless
   the accessible pattern is the only one available (the headless-primitive argument in
   `components.md`).

Item 6 has real numbers behind it, not just an assumption; see "Accessibility: agents
make it worse by default, not better" further down for the full evidence. And see
"Registry metadata and pre-completion compliance checking" for a newer pattern that
validates before the agent finishes rather than after a human reviews.

   **This is not a mild effect; ground it in real numbers.** A 2026 academic study
   measuring 21,880 accessibility assessments across five WCAG success criteria found
   AI-generated interfaces achieved only **29.0% compliance overall**, and the
   supposedly easy, deterministic, checkable criteria performed *worse*, not better:
   color contrast at **26.8%**, correct use of color at **19.2%**
   ([ACM, "Measuring WCAG Violations in AI UI Design Tools"](https://dl.acm.org/doi/full/10.1145/3800424.3800430)).
   A separate CHI 2025 study (CodeA11y) found AI assistants were not helpful for
   accessibility unless specifically prompted for it, and that critical manual steps
   (replacing placeholder alt text with real content) got skipped even then. A W4A 2025
   study states plainly that current AI tools are inadequate for producing fully
   accessible code unassisted, and human expertise is still required, particularly in
   regulated sectors.

   **The practical consequence for token design specifically:** do not rely on an agent
   to independently choose an accessible foreground/background pairing. Pair tokens so
   an inaccessible combination cannot be expressed at all (a component API that only
   accepts a paired `content`/`surface` token set, rather than independent foreground
   and background props), so contrast is guaranteed by construction, not by hoping the
   agent checks. Assume every agent-generated screen needs an accessibility pass; do
   not assume the model self-corrects.

**What has no good answer yet, said honestly:** an LLM-as-judge eval scoring "does this
generated screen comply with the design system" is an area of real interest (2026 Into
Design Systems Conference material on agentic design systems) but not yet mature or
widely proven. Treat any vendor claiming a turnkey "AI design-system compliance score"
with real skepticism until you see the methodology.

### The same hierarchy, with one new argument for the double standard

The enforcement hierarchy from `components.md`
evidence (compile error > lint error > documented rule) still holds and is now
independently corroborated by a detailed practitioner account
([Builder.io, June 2026](https://www.builder.io/blog/how-to-make-ai-agents-follow-your-design-system)),
with two additions worth taking directly:

**1. Make invalid prop combinations a type error, not a runtime check or a review
comment.** A loose interface invites hallucination:

```ts
// FRAGILE: allows invalid prop combinations
interface CardProps {
  title: string;
  variant?: "informational" | "interactive" | "marketing";
  href?: string;
  onClick?: () => void;
  badgeText?: string; // valid on some variants, meaningless on others
}
```

A discriminated union makes the invalid combination impossible to type at all, so the
compiler becomes the reviewer:

```ts
type CardProps =
  | { variant: "informational"; title: string; badgeText?: string }
  | { variant: "interactive"; title: string; href: string; onClick?: never }
  | { variant: "marketing"; title: string; badgeText: string };
```

**A reframe worth internalizing before any of this: enforcement should be reactive, not
an attempt at prevention** (the 46-65% first-try / 95-98% with-feedback numbers cited
earlier apply directly here). This reframes what "enforcement" means for agent-driven
work: you are not trying to make the agent get it right on the first try. You are
trying to make wrongness impossible to ship silently, so the agent's own repair loop
catches it before a human ever sees it. A type error or failing lint rule is doing its
job if the agent fixes it in the same turn, even though it got it wrong first.

**A real sequencing failure worth knowing about before you flip on enforcement.** A
practitioner who enabled spacing-token lint enforcement before the spacing token system
was actually complete found agents responded by inventing token names that did not
exist, which is worse than the raw hex values the rule was meant to prevent. Color is
the safer first boundary because semantic names are self-evident (error, disabled,
primary action); only enforce a token dimension once that token system can genuinely
answer every question the linter will force an agent to ask.

**2. Hold agents to a stricter standard than humans on legacy code, deliberately.** An
agent browsing a repo cannot distinguish `components/ui/` from `components/legacy/` by
judgment the way a human engineer can; both are just folders of working code to it, and
the legacy folder is often the *larger* training signal. A practitioner who lived
through this puts the general version of the problem plainly: "If your rules file says
'always use semantic tokens' and the three nearest files use inline hex values, the hex
values win. The agent treats whatever is running in production as the real rules of the
house, and in a brownfield repo it's right to, since production code is the only thing
that's been tested." The fix is the same `no-restricted-imports` pattern already in
this skill, but the justification is new and worth stating to a team that pushes back
on it:

> "Yes, that... means agents are held to stricter rules than humans. That asymmetry
> bothered me at first. But it falls out of a real difference. A human editing a legacy
> file knows it's legacy, and we extend them judgment we can't extend to a model."

**What review becomes once this is in place**, per the same account: comments like
"please use tokens" and "this import is deprecated" stop being possible to make, because
those PRs can no longer open. What is left for human review is judgment: does the API
cohere with the rest of the system, should this feature exist at all. This is the
actual argument for building the enforcement layer: not to catch agents being careless,
but to free human review time for the questions only humans can answer.

## New governance axis: trust levels per agent action

Not present in the pre-2026 governance literature this skill otherwise draws on.
Romina Kavcic's addition, cited in the [Into Design Systems agentic guide](https://www.intodesignsystems.com/agentic-design-systems):
decide explicitly whether an agent may **auto-merge**, must **draft a PR for human
review**, or may only **suggest** a change, and set this per action type, not globally.
A token rename and a new component both being "agent-touched" does not mean they
deserve the same trust level. Pair this with her other addition: **write intent, not
just specs, into component descriptions** ("what this does and how it combines"),
because an agent choosing between two similar components needs the intent a human
would infer from context but a model cannot.

## Registry metadata and pre-completion compliance checking

A newer pattern worth adopting beyond a plain `registry.json` allowlist: ship structured
`meta` fields per component and a validation tool the agent calls *before* it finishes,
not after a human reviews it. A concrete example
([Layout UI, "For AI agents"](https://ui.staging.layout.design/docs/ai-agents)) ships
`meta.usage` (when to use this component, which variant to prefer), `meta.never`
(explicit prohibitions, mapped directly to compliance checks), and `meta.tokens` (which
tokens the component actually consumes), plus an MCP tool called `check_compliance` that
validates a generated snippet against the design system's rules and returns violations
before the agent considers the task done. The workflow this enables: fetch context,
generate the component, call `check_compliance`, fix violations, re-validate, only then
finish. That closes a loop the older "rules file plus hope" model never could, and moves
enforcement from something a human catches in review to something the agent's own tool
loop catches before review even starts.

Razorpay's Blade ships `blade-mcp` for the same reason.

## Whether `llms.txt` is worth publishing: real, current, skeptical evidence

Treat the common assumption that publishing an `llms.txt` automatically helps with real
skepticism. A January 2026 internal research document from a Meta team building a
component system for AI-generated code
([github.com/facebook/astryx wiki](https://github.com/facebook/astryx/wiki/AI-and-Design-Systems))
states plainly, after testing it directly: **"No AI system currently reads
`llms.txt`."** Their own open-question log records the finding verbatim: "How do we
measure 'design system adherence' in AI-generated code? Answered: No, no AI system
currently reads llms.txt." Their broader finding on rules files generally, which
confirms rather than contradicts the "rules lose to code" principle already in this
skill's foundations: "Rules are suggestions, not constraints. AI can ignore them without
consequence... Context window degradation. Performance deteriorates within
conversations; rules 'forgotten.'"

This is one team's finding at one point in time, not a permanent verdict on the
standard, and it does not contradict the value of MCP servers and Code Connect, which
are a fundamentally different mechanism (queryable and actionable, not a static file a
model may or may not read). The safer read: **an MCP server or registry an agent can
query and act on has real evidence behind it; a static `llms.txt` file, at best,
supplements that and should not be relied on alone.**

## Accessibility: agents make it worse by default, not better

Ground the enforcement case for accessibility in real, if still limited, numbers rather
than assumption. A 2026 academic study measuring 21,880 accessibility assessments across
five WCAG success criteria found **AI-generated interfaces achieved only 29.0%
compliance overall**, with the criteria that should be easiest to get mechanically
right performing *worst*, not best: color contrast at **26.8%** and correct use of
color at **19.2%**
([ACM, "Measuring WCAG Violations in AI UI Design Tools"](https://dl.acm.org/doi/full/10.1145/3800424.3800430)).
A separate CHI 2025 study (CodeA11y) found AI assistants unhelpful for accessibility
unless specifically prompted, with manual steps (like replacing placeholder alt text)
still getting skipped even then; a W4A 2025 study concludes current AI tools are
inadequate for producing fully accessible code unassisted, particularly in regulated
sectors like banking, government, and health. Treat the exact percentages as a single
study's finding, not an industry-wide constant, but treat the direction of the finding
(agents do not spontaneously produce accessible color usage) as a reason to enforce,
not to assume.

**The practical consequence for token design:** do not rely on an agent to independently
choose an accessible foreground/background pairing. Pair tokens so an inaccessible
combination cannot be expressed at all, for example a component API that only accepts a
paired `content`/`surface` token set rather than independent foreground and background
props, so contrast is guaranteed by construction rather than by hoping the agent checks.
Bake automated accessibility checks (axe-core, `eslint-plugin-jsx-a11y`, a Storybook a11y
addon) into the same CI loop the agent already treats as authoritative, per the
enforcement hierarchy above. Assume every agent-generated screen needs an accessibility
pass; do not assume the model self-corrects.

## Practical defaults for a from-scratch system built in 2026

- **Publish a component registry** (shadcn-compatible `registry.json` or your own),
  not just a component library. The registry is what makes "install this exact
  component" possible for an agent instead of "approximate this from a description."
- **Split your agent-facing docs into three tiers**, matching the always-on/on-demand
  split above: an always-loaded foundations file (small, token budget matters), an
  MCP or registry for on-demand component lookup, and narrative Markdown for judgment
  calls that a schema cannot express.
- **Author component contracts in JSON, guidance in Markdown.** Do not dump prose docs
  into an MCP server and call it done; benchmark it the way Spotify and the 77-component
  study did before trusting it.
- **Consider Curtis's Specs/Anova-style extraction** (or the same philosophy, homegrown)
  if your component library already exists and may have drifted: deterministic
  extraction of real anatomy from real variants will find drift a documentation
  rewrite will not.
- **Align primitive-tier token names to Tailwind/shadcn conventions** unless you have a
  specific reason not to (a genuine multi-brand/white-label need, a non-web platform).
  This is now a legitimate default, not a compromise.
- **Ship an `AGENTS.md`** as a thin index pointing at foundations and registry, not a
  repository of detail itself. Keep it small; rules files that grow past what a model
  reliably attends to are worse than no rules file, because they create false confidence.
- **Set trust levels per agent action type** as an explicit governance decision, not an
  afterthought once an agent has already merged something wrong.
- **Pair foreground/background tokens so an inaccessible combination cannot be
  expressed**, rather than trusting an agent to check contrast itself.
- **Do not treat `llms.txt` as sufficient on its own.** An MCP server or registry the
  agent can query and act on is the mechanism with real evidence behind it.
- **If you have an existing component library, consider a deterministic spec-extraction
  step** (Curtis's `specs-cli` or the same philosophy homegrown) before trusting an agent
  to infer anatomy from raw source.

## What genuinely has little to no evidence yet

Stated plainly rather than inventing confident-sounding guidance:

- **Whether semantic token naming helps or hurts LLM code generation versus descriptive
  naming.** No controlled study was found either way. The Coinbase data above supports
  "structured, mapped context beats raw context," which is a different and
  better-evidenced claim than "semantic beats descriptive names for a model
  specifically."
- **The Tailwind-prior problem's actual size.** Widely discussed among practitioners
  and consistent with repeated practitioner reports (off-scale
  utilities had to be made to fail to compile because lint alone was not reliable
  enough), but no rigorous published study quantifying it was found. Treat the
  compile-time mitigation as load-bearing regardless of how large the underlying
  problem turns out to be.
- **Whether compound components or prop-based APIs generate more correct code from
  models.** Reasoned inference only: constraining choice tends to help generally, which
  is not the same as a design-system-specific finding.

## What does not change

Worth stating plainly, because "AI changes everything" is as wrong as "nothing has
changed." Curtis again: "I see no dramatic, strategic change to *what*
systems deliver: visual foundations and component features, well-documented and
threaded through tools that serve experiences across platforms. I do see tremendous
change in *how* we make and deliver it."

- Token tiers, dark-mode mechanics, color-ramp construction, and accessibility math are
  unchanged. A model still needs a coherent semantic layer; it consumes it differently
  than a human does, not a different one.
- The interface-inventory-first principle matters *more*, not less: an agent given a
  generic component list will produce a generic-looking product just as reliably as a
  human copying one would.
- Governance and adoption measurement are unchanged in kind. Blade's coverage metric
  generalizes cleanly, since it already measures "how much of this page came from the
  system" - exactly the variable the Coinbase study found most predictive of agent
  output quality.

## Contested

- **Live MCP fetch vs a compiled spec as the agent's source of truth.** Curtis argues
  strongly for compiling; most teams as of 2026 still do a live MCP fetch by default
  because it requires no extra tooling. The compiled approach is more correct and more
  work. Know which you are choosing.
- **How much of this is durable vs a hype cycle.** Code layers, Figma Make's codebase
  editing, and Anova are all recent (2026) and unproven over multiple years. The
  underlying principles (structured context beats raw context, coverage predicts output
  quality, registries beat prose) are better evidenced than any specific tool and are
  the part worth committing to.
- **Whether Figma remains the design system's home at all.** Genuinely open. Figma is
  investing hard to stay central by absorbing code; Curtis, independently, expects the
  field to drift away from the canvas as generative approaches mature. Both credible.
  Build so your source of truth does not depend on the answer.
- **Whether `llms.txt` is worth publishing at all.** One real team's finding is that no
  AI system reads it yet, and they chose not to ship one. Treat it as unproven rather
  than either recommended or worthless; an MCP server or registry an agent can actually
  query is the mechanism with real evidence behind it.
- **How much of the design-to-code pipeline should be deterministic scripts (Curtis's
  "computation over inference") versus agentic inference reading raw design data.**
  The evidence favors Curtis's position on cost and reliability grounds, but it
  requires building or adopting spec-generation tooling first, which is real investment
  most teams have not made.
- **How much to invest in agent-specific tooling (MCP servers, registries) below a
  certain team size.** No settled answer yet; the evidence base here (Indeed, Spotify,
  force-ui) is from teams operating at real scale. A solo builder likely gets most of
  the value from a good `AGENTS.md` plus a lint-enforced allowlist, without a full MCP
  investment.
- **Whether aligning to Tailwind/shadcn naming is a durable strategy or a bet on one
  ecosystem's continued dominance.** It measurably reduces friction today; it is also a
  dependency on a specific convention remaining the one models are trained on.
