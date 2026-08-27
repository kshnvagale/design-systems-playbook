# Design Systems Playbook

An agent skill for designing and building design systems from scratch, for products at
10,000+ users spanning B2B, B2C, and internal tools.

Built for 2026 and beyond: AI agents write a meaningful share of UI code, designers work
increasingly in code, and the design tool is a view onto the system rather than its
database.

## Use

`SKILL.md` is the entry point. It routes to `references/` on demand rather than preloading
everything, so a typical task loads the map plus one or two references.

```
1. DISCOVER   references/discovery.md          what to ask before deciding anything
2. DECIDE     references/tokens.md             tiers, dark mode, theming, pipeline
              references/color.md              ramps, OKLCH, WCAG, palette restraint
              references/visual-language.md    type, grid, icons, motion, delight, content
              references/naming.md             token taxonomy, ARIA-aligned component names
              references/components.md         canonical inventory, build order, API design
3. BUILD      references/orchestration.md      fan out to subagents, verify on return
              references/build-architecture.md file shape, layer boundaries, verify gate
              references/ai-agents.md          agent-consumable delivery and enforcement
4. DELIVER    references/delivery.md           HTML preview, approval, Storybook, skill file
5. GOVERN     references/governance.md         team models, adoption, why systems die
6. VERIFY     references/evaluation.md         does it work, and is it sufficient
```

## The order that matters

Discovery is not optional and comes first. The HTML preview comes **before** React and
carries the **complete** component inventory, because that is where completeness is proven.
Approval gates the React work. Enforcement exists before anything is generated.

```
discover -> inventory -> visual language -> tokens -> naming
         -> HTML preview (complete, interactive) -> APPROVAL
         -> foundation choice -> React -> enforcement -> Storybook
         -> generated skill file -> ship -> held-out test
```

## Output

A working design system in code, not a strategy document: DTCG tokens with a deterministic
build script, an interactive single-file HTML preview covering the full inventory,
components on headless primitives, a registry allowlist, CI enforcement (raw values, layer
boundaries, spec existence, token drift), a published Storybook with a story per variant and
state, and a generated skill file so the system can be used without rediscovering it.

## Opinions worth knowing before you use it

- **Central capacity is non-optional.** Nathan Curtis publicly retracted his own influential
  framing that favored federated teams. Federated is a facet you add later, never a model
  you choose.
- **Delegate by default.** Past roughly 6 components, building serially is a failure mode.
- **Generate only the color you need.** Availability is permission.
- **shadcn/ui is the default foundation**, with Astryx read as a spec rather than taken as a
  dependency. Training-data familiarity dominates agent codegen quality.
- **Gates prove discipline, never sufficiency.** Only building a real screen tells you the
  system is complete.

## Sources

Primary sources throughout: Nathan Curtis / EightShapes, Brad Frost, Radix, Material 3, IBM
Carbon, Shopify Polaris, Adobe Spectrum, GitHub Primer, Atlassian, Razorpay Blade, Meta
Astryx, W3C DTCG, WAI-ARIA APG, Figma's Code Connect evals, Sparkbox survey data, and
published practitioner postmortems. Contested points are flagged where experts genuinely
disagree, and single-study findings are labeled as such.

## Maintenance

`references/ai-agents.md` is the most time-sensitive file. Its tooling claims, survey data,
and product specifics should be re-verified periodically. The underlying principles
(structured context beats raw context, coverage predicts output quality, registries beat
prose) are more durable than any tool named in it.

Not yet validated end to end on a full build. Discovery has been cold-tested; the
token-compilation and Storybook phases are where gaps are most likely to surface first.
