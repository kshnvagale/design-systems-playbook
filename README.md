# Design Systems Playbook

An agent skill for designing and building design systems from scratch, for products at
10,000+ users spanning B2B, B2C, and internal tools.

Built for 2026 and beyond: AI agents write a meaningful share of UI code, designers work
increasingly in code, and the design tool is a view onto the system rather than its
database.

## How it loads

`SKILL.md` is the entry point. It routes to `references/` on demand rather than preloading,
so cost scales with what you actually use:

| Level | Loads | Cost |
|---|---|---|
| Metadata (name, description) | Always, in the system prompt | ~120 tokens |
| `SKILL.md` | When the request matches | ~6,000 tokens |
| A reference file | Only when routed to and read | 0 until opened |

A typical task lands around 9-13k tokens. All 15 files at once is ~78k, which only happens
on a full end-to-end build.

## Map

```
1. DISCOVER   discovery.md            what to ask before deciding anything, including
                                      brand color, component style refs, logo, motion

2. DECIDE     tokens.md               tiers, dark mode, theming, pipeline
              color.md                hue count, ramp depth, the three-palette model
              visual-language.md      type, grid, icons, motion, delight, content
              naming.md               token taxonomy, ARIA-aligned component names
              components.md           canonical inventory, build order, API design
              marketing-surfaces.md   landing pages, the brand/product seam

3. BUILD      orchestration.md        fan out to subagents, verify on return
              build-architecture.md   file shape, layer boundaries, the verify gate
              ai-agents.md            agent-consumable delivery and enforcement

4. DELIVER    preview.md              two previews, two approval gates
              handoff.md              Storybook, PROGRESS.md, generated skill file

5. GOVERN     governance.md           team models, adoption, why systems die

6. VERIFY     evaluation.md           does it work, and is it sufficient
```

## The order that matters

Discovery is not optional and comes first. The HTML preview comes **before** React and
carries the **complete** component inventory, because that is where completeness is proven.
Two gates, not one: the system is approved before page templates are even discussed.

```
discover -> inventory -> visual language -> tokens -> naming
         -> PREVIEW 1: base system, complete inventory  -> GATE 1
         -> ask about templates, collect page list and real content
         -> PREVIEW 2: templates only                   -> GATE 2
         -> ask: marketing in scope? which foundation?
         -> React -> enforcement -> Storybook -> generated skill file
         -> ship -> held-out test
```

## Output

A working design system in code, not a strategy document: DTCG tokens with a deterministic
build script, an interactive single-file HTML preview covering the full inventory,
components on headless primitives, a registry allowlist, CI enforcement (raw values, layer
boundaries, spec existence, token drift, semantic headings, anchor integrity), a published
Storybook with a story per variant and state, a `PROGRESS.md` tracking every item, and a
generated skill file so the system can be used without rediscovering it.

## Opinions worth knowing before you use it

- **Central capacity is non-optional.** Nathan Curtis publicly retracted his own influential
  framing that favored federated teams. Federated is a facet you add later, never a model
  you choose.
- **Delegate by default.** Past roughly 6 components, building serially is a failure mode.
- **Decide hues, then take the whole ramp.** Roughly 60-85 primitives. Truncating a ramp is
  worse than generating it, because a missing step gets improvised into existence.
- **Three palettes, one foundation.** UI, categorical, and illustration optimize for
  different things and must not be merged.
- **shadcn/ui is the default foundation**, with Astryx read as a spec rather than taken as a
  dependency. Training-data familiarity dominates agent codegen quality.
- **Gates prove discipline, never sufficiency.** Only building a real screen tells you the
  system is complete.

## Sources

Primary sources throughout: Nathan Curtis / EightShapes, Brad Frost, Radix, Material 3, IBM
Carbon, Shopify Polaris, Adobe Spectrum, GitHub Primer, Atlassian, Razorpay Blade, Meta
Astryx, Indeed, W3C DTCG, WAI-ARIA APG, Figma's Code Connect evals, Sparkbox survey data,
and published practitioner postmortems. Contested points are flagged where experts genuinely
disagree, and single-study findings are labeled as such.

## Maintenance

`references/ai-agents.md` is the most time-sensitive file. Its tooling claims, survey data,
and product specifics should be re-verified periodically. The underlying principles
(structured context beats raw context, coverage predicts output quality, registries beat
prose) are more durable than any tool named in it.

**Known weakness:** numbers that appear in more than one file drift when one is corrected.
`color.md` is the single source for palette sizing and `components.md` for the inventory;
other files should defer rather than restate. This has already caused one real failure.

**Not yet validated end to end on a full build.** Discovery has been cold-tested several
times. The token-compilation and Storybook phases are where gaps are most likely to surface
next.
