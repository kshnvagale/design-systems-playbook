---
name: "design-systems-playbook"
description: "Use when starting, planning, auditing, or reviewing a design system: token architecture, light/dark theming, color palettes, naming conventions, which components to build, governance, and how to make the system consumable and enforceable for AI coding agents. Built for products at 10,000+ users spanning B2B, B2C, and internal tools. Read before proposing any design-system structure, and before advising a designer of any level."
---

# Design Systems Playbook

## Work in three phases. Do not skip phase 1.

```
1. DISCOVER  ->  establish context you cannot observe      references/discovery.md
2. DECIDE    ->  make the calls, using the tables below    references/*.md
3. VERIFY    ->  test that it works and is sufficient      references/evaluation.md
```

**The most common failure of an agent using this skill is jumping straight to phase 2.**
A token architecture proposed before you know whether the product is multi-brand,
cross-platform, or regulated is a guess wearing the costume of expertise.

If the user has not given you that context, ask. One batch of questions, with your
proposed defaults attached so they can reply "defaults are fine." `discovery.md` has the
eight questions that actually change the architecture, the defaults to assume when you
get no answer, and the things you should decide yourself rather than ask about.

---

## What you actually deliver

**A working design system in code, not a document about one.** Documentation is a
byproduct, generated from the source where possible. If the output is a slide deck, a PDF,
or a design file nobody imports, you have produced a design system that does not exist.

The artifact set, in build order:

| Phase | Artifacts | Done when |
|---|---|---|
| **Discovery** | Written context summary, corrected by the user. Visual philosophy in 3-5 falsifiable sentences. Stated assumptions. | The user has corrected at least one line of it |
| **Foundations** | `tokens.json` (DTCG). A **deterministic build script** compiling it to CSS custom properties and any platform outputs. Color ramps, type roles, spacing, radius, motion, elevation. Contrast audit across every token pairing in every theme. | Contrast audit passes; no component references a raw value |
| **Preview** | **One self-contained HTML file with the COMPLETE inventory**, sectioned by atomic layer, fully interactive, motion running, every variant and state, theme toggle, real content, the company logo, plus 2-3 composed screens. | Every component that will exist is visible and judgeable |
| **Approval** | An explicit yes on look, feel, motion, and completeness. | Recorded. **React work does not start before this.** |
| **Components** | React components on headless primitives, uniform file shape per component, every interaction state, `registry.json` allowlist. | A screen can be built from them without inventing anything |
| **Storybook** | A story file per component: CSF3, `autodocs`, `argTypes` with controls on every prop, one story per variant and per state, `play` functions for interactions, theme switcher, a11y passing. Published to a URL. | Every component is explorable and interactive by someone who did not build it |
| **Enforcement** | Lint rules (no raw values, registry-only imports). Off-scale utilities that fail to compile. CI wiring. A fixture test per rule. Automated a11y checks. | CI fails on a raw hex, and every rule has been observed to fire |
| **Docs** | A page per component: anatomy, live example, props table, all states, a11y notes, content guidance, do/don't. | A new engineer can use a component without asking anyone |
| **Generated skill file** | **One file**, part generated (allowlist, tokens, scales, so it cannot drift) and part hand-written (philosophy, judgment calls). States the Storybook URL as source of truth and the never-invent rules. Also shipped as `AGENTS.md`. | Regenerating is one command, and a cold agent builds a compliant screen from it |
| **Verification** | Held-out test result with an objective score. A gap inventory. Coverage baseline instrumented. | You know what your system is missing, in writing |

**Definition of done for v1:** someone who was not involved in building it can produce a
new screen entirely from the system, CI blocks a raw hex, the held-out test scores above
90% component reuse with zero invented components, and coverage is being measured in
production.

**Scope honestly.** If the user asked for a plan, deliver phases 1-2 plus a roadmap and
say so. If they asked for a design system, the default is the full set. Never let the
engagement end at Discovery and call it a design system, and never produce documentation
for components that do not exist.

---

## Routing

| Read this | When you are deciding |
|---|---|
| `references/discovery.md` | What to ask before anything else. Start here. |
| `references/tokens.md` | Tiers, dark mode mechanics, theme axes, pipeline, non-color tokens |
| `references/color.md` | Ramp construction, OKLCH, WCAG/APCA, palette composition |
| `references/visual-language.md` | Typography, grid and responsive, iconography, motion, state design, content and voice |
| `references/naming.md` | Token taxonomy, component names vs ARIA, variant and prop naming |
| `references/components.md` | Canonical inventory, interface inventory method, build order, API design |
| `references/build-architecture.md` | Uniform component file shape, layer dependency rules, the verify pipeline |
| `references/governance.md` | Team models, contribution, adoption, why systems die |
| `references/ai-agents.md` | Agent-consumable delivery, enforcement, specs-as-data, Figma's real position |
| `references/orchestration.md` | Running the build with subagents: what to parallelize, ownership, verification |
| `references/delivery.md` | HTML preview spec, approval gate, Storybook requirements, the generated skill file |
| `references/evaluation.md` | Whether the system works, and whether it is sufficient |

Read the file for the decision in front of you. Do not preload all of them.

---

## Five rules that override everything below

**1. A design system is downstream of the product.** Never start from another system's
component list. Start with an inventory of real screens. Copy architecture, not catalogs.

**2. A design system is a product, not a project.** Vision, roadmap, release process,
support channel, success metric. Without those it stalls the moment its founding team
gets reassigned.

**3. Ship a small real slice before architecting the whole thing.** Almost every failure
mode traces back to building the complete taxonomy before anyone outside the core team
used any of it.

**4. Enforcement beats documentation, and it is not optional once agents write code.** An
agent inventing a component does not throw an error. It produces working, type-checking,
correctly-rendering code that quietly is not your design system. If a rule exists only in
prose, it does not exist. `ai-agents.md`

**5. Gates prove discipline, never sufficiency.** Every automated check asks "did you use
the system correctly." None asks "was the system enough." Only building a real screen and
comparing it side by side answers that. `evaluation.md`

**6. Delegate by default. This is not optional.** For any build past roughly 6 components
or 8 files, **you MUST run as an orchestrator and fan work out to subagents.** Doing it
serially in one session is a failure mode: it is slower, it burns your context on
implementation detail, and quality degrades on the later components because the context is
full by then. Announce the plan before dispatching. The orchestrator keeps the user
conversation, the token source, the registry, naming, and every decision. Subagents get
**exclusive file ownership, the spec inline, and an explicit return format.** Verify the
artifacts on return, never the self-report. `orchestration.md`

**7. Lead with design, land in code.** A system built only from architecture and
enforcement will be correct and generic. Decide the visual language deliberately
(`visual-language.md`), then express every decision as a token, prop, or documented
constraint. The test for any craft guidance: **could a developer or an agent implement
this without a follow-up question?** "Generous whitespace" fails. "24px between form
fields" passes.

---

## Order of operations

1. **Discovery.** Context, constraints, who maintains it, who writes the code.
2. **Interface inventory** of the live product, cross-disciplinary, time-boxed.
3. **Visual language.** Type roles and scale, grid and breakpoints, density per surface,
   icon grid and stroke, motion intent, voice. Do this *with* tokens, not after.
4. **Primitive plus semantic tokens, light mode first.** ~40-80 primitives, ~60-120 semantic.
5. **Lock naming.** Renaming after adoption is a breaking change.
6. **Dark mode as a second authored value set**, never an inversion. Needed before the
   preview, because the preview must ship a working theme toggle.
7. **Single HTML preview containing the COMPLETE component inventory**, organized in
   atomic layers (foundations, atoms, molecules, organisms, layout primitives, pages),
   fully interactive with motion running. **Then get explicit approval.**
   This is where completeness is proven. If a component is not in the HTML, it will be
   forgotten. `delivery.md`
8. **React components**, on a headless primitive library, not from scratch. Only after the
   preview is approved. `components.md`, `build-architecture.md`
9. **Enforcement layer before generated screens.** Registry, lint, layer boundaries,
   off-scale values that fail to compile. Generation before enforcement amplifies drift.
10. **Storybook**, one story file per component, published to a URL.
11. **Generate the system's own skill file** so designers and agents can use it without
    rediscovering it. `delivery.md`
12. **Ship, instrument coverage, iterate.**
13. **Held-out test.** Fresh agent, screenshot from a *different* product, existing
    components only. The deliverable is the inventory of what your system lacks.

Dark mode is step 7 for authoring, but step 4 must assume it exists. Retrofitting onto
components that consumed primitives means re-touching every state.

---

## Fast decision table

| Question | Default |
|---|---|
| How many token tiers? | Three in the data model. Expose only two to designers. |
| How many tokens? | ~40-80 primitive, ~60-120 semantic. Multi-brand grows the primitive layer, not the semantic one. |
| Dark mode now or later? | Semantic layer designed for it now. Author values whenever. |
| Dark base color? | Dark grey (`#121212` class), not pure black. True black only for a stated OLED need. |
| Color ramp steps? | 10-12, each with a documented purpose. Steal Radix's step mapping. |
| Color space? | OKLCH/LCH/HCT. Never build a ramp by eye in HSL. |
| How many type roles? | 8-12 composite tokens, each bundling family, size, weight, line height. Not separate atomic tokens. |
| Type scale ratio? | 1.125-1.2 for dense product UI. Larger ratios waste vertical space. Round to whole pixels. |
| Emphasis via weight or color? | Weight. Color expresses hierarchy, not importance. Pick one and document it. |
| Breakpoints? | Derive from where content breaks, not device names. Carbon's 320/672/1056/1584 is a safe default. |
| Density? | Per surface, not global. Scanning many things = tight. Reading about one thing = roomy. |
| Icon grid? | 24x24 default, 16x16 for dense dashboards. One stroke weight, squared terminals, optical not mathematical sizing. |
| Icon without a label? | Only if universally understood. Decorative icons get `aria-hidden`, meaningful ones need an accessible name. |
| Motion personality? | Ask, do not assume. Productive/standard by default, with a named set of expressive moments. Never leave it silent. |
| Where can delight go? | Onboarding, empty states, success, micro-feedback. **Never** on an error, decline, or destructive confirm. |
| Adding a microinteraction? | Name the uncertainty it resolves for the user. If you cannot, it is decoration. Feedback weight matches stakes. |
| Ask for the logo? | **Always, explicitly.** SVG, all lockups, and a dark-background version. A one-color logo breaks in dark mode. |
| Storybook before or after approval? | After. Preview in one HTML file, get a yes, then build Storybook. |
| Component done when? | It has a story per variant and per state, controls on every prop, a `play` function if interactive, and passes a11y. |
| Where does the system live? | The published Storybook URL. Design tools and agents fetch from it and never re-invent. |
| Use subagents? | **Yes, by default,** past ~6 components or ~8 files. Serial building is a failure mode, not a safe choice. |
| Parallelize what? | Components, stories, research, audits. **Never** tokens, registry, naming, or the preview. |
| What goes in the HTML preview? | **Everything.** The full inventory by atomic layer. Deferring a component in the React build order never means omitting it here. |
| Which components are required? | The canonical inventory in `components.md`: 24 atoms, 27 molecules, 16 organisms, 6 layout primitives. Domain components on top. |
| Two agents, one file? | Never. If two could write it, only the orchestrator writes it. |
| Trust a subagent's report? | No. List the files, check `git status`, re-run the gates, spot-read one file in full. |
| Batch size? | 3-5. Expect partial failure and make every task independently retryable. |
| Depth: borders or shadows? | Ask. It decides whether elevation is a real token axis, and it interacts with dark mode. |
| Illustration style? | Ask. Decorative imagery gets `aria-hidden`, and no information may live only in an image. |
| Motion duration? | 50-200ms feedback, 250-400ms transitions, 450-600ms large. Every token needs a reduced-motion answer. |
| Skeleton or spinner? | Skeleton when you know the shape, spinner when you know only the wait. A mismatched skeleton is worse than none. |
| Define voice how? | Four or five traits phrased as tensions ("helpful but humble, not arrogant"), each with a Don't/Do table. Adjectives alone are useless. |
| WCAG or APCA? | Ship WCAG 2.x. APCA is directional, not normative. |
| Token separator? | Dot in the definition tree, kebab in CSS, camelCase in JS. |
| Modal or Dialog? | `Dialog`. Matches the ARIA role. Modal is a behavior. |
| Dropdown? | Not a pattern. Decide Listbox, Menu, or Combobox, then name it that. |
| Chip or Badge? | Interactive with selected state = Chip. Passive metadata = Badge. Never ship both plus Tag plus Pill. |
| Data Table on day one? | No. Top-3 time sink with Date Picker and Combobox. |
| Hand-roll an accessible Combobox? | No. Radix Primitives or React Aria. |
| Central or federated team? | **Central first, always.** Federated is an optional later addition, never a starting choice. |
| Measure adoption how? | Coverage (% of nodes from the system), not import counts. |
| Buy zeroheight/Supernova? | Not below ~20 people. Storybook plus MDX covers it. |
| Figma or code as source of truth? | Code, or a platform-neutral spec. The design tool is a view onto it. |
| Expose the system to an agent how? | Foundations always-on, components on-demand via registry/MCP. Never one alone. |
| JSON or Markdown for docs? | JSON for the API contract, Markdown for narrative judgment. |
| Custom token names or Tailwind-aligned? | Align at the primitive tier unless you have a reason not to. |
| Is `llms.txt` enough? | No. Use a registry or MCP the agent can query and act on. |
| Stop agents shipping bad contrast? | Pair foreground/background tokens so an invalid combination cannot be expressed. |

---

## Precedent: what to take from which system

| System | Take this |
|---|---|
| **Radix Colors** | The 12-step ramp with a documented purpose per step. The most reusable artifact in the field. |
| **Material 3** | Dark-mode methodology: lightness encodes elevation, the contrast headroom rule, tonal surfaces. |
| **Blade (Razorpay)** | Coverage as the adoption metric. 1:1 Figma-prop to code-prop naming. Written API decision records. |
| **Atlassian** | Motion tokens named by intent (`motion.popup.enter`), not by raw duration values. |
| **Adobe Spectrum** | The architecture split: React Aria as the interaction engine, Spectrum as the visual layer. |
| **Shopify Polaris** | B2B admin reference: data tables, filters, bulk actions, one token source to many outputs. |
| **IBM Carbon** | Density and restraint defaults for dense enterprise software. |
| **GitHub Primer** | Proof that a single-product system should stay simple. Match complexity to portfolio breadth. |

Universal agreement across all of them: three token tiers, dark mode as authored values
rather than inversion, headless primitives underneath, ARIA-aligned naming, and adoption
measured rather than assumed.

---

## Failure modes

- Building 60 components nobody adopts, copied from a library instead of derived from an
  inventory.
- Retrofitting dark mode onto components that hard-coded primitives.
- Exposing every token tier to every designer. One reported case: ~1,500 tokens for a
  single text field.
- Skipping the semantic tier, which is what makes token counts balloon with no clean way
  to theme.
- Picking a brand hue without checking it can serve as an accessible interactive color.
- Building the UI palette and the data-viz palette as one thing.
- Assuming light-mode contrast compliance transfers to dark.
- Zero escape hatches (teams fork silently) or unlimited ones (every screen reinvents the
  palette). The line: content composition free, visual overrides lint-blocked.
- Trusting a gate that has never failed. Fixture-test every rule before believing it.
- Trusting an adoption score without validating the scorer on a known answer.
- Believing a 100% reuse score means the system is complete. It measures discipline.
- Writing a rule the format cannot express, which generates noise and discredits the gate.

---

## Contested calls you are making whether you notice or not

Each reference file has a Contested section. The ones that come up most:

- **2-tier vs 3-tier tokens.** Resolve: 3 in the data model, 2 exposed.
- **WCAG 2 vs APCA.** Resolve: WCAG 2 is the compliance floor.
- **True black vs dark grey.** Resolve: dark grey. Apple defaults to pure black, so this
  is a real platform disagreement, not settled fact.
- **Props vs slots.** Mid-swing toward composition. Know which side you chose.
- **How permissive escape hatches should be.** Genuinely unresolved.
- **Whether contribution is worth pursuing.** Atlassian restricts it to fixes only;
  others invest heavily.
- **Whether the design tool remains the system's home.** Open. Build so your source of
  truth does not depend on the answer.
- **Live MCP fetch vs a compiled spec.** Compiled is more correct and more work.
