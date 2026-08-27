# Design Systems Playbook

An agent skill for designing and building design systems from scratch, for products at
10,000+ users spanning B2B, B2C, and internal tools.

Built for 2026 and beyond: AI agents write a meaningful share of UI code, designers work
increasingly in code, and the design tool is a view onto the system rather than its
database.

## Use

`SKILL.md` is the entry point. It routes to `references/` on demand rather than
preloading everything.

```
1. DISCOVER  ->  references/discovery.md      what to ask before deciding anything
2. DECIDE    ->  references/*.md              tokens, color, visual language, naming, components
3. BUILD     ->  references/orchestration.md  fan out to subagents, verify on return
4. DELIVER   ->  references/delivery.md       HTML preview, approval, Storybook, generated skill
5. VERIFY    ->  references/evaluation.md     does it work, and is it sufficient
```

## Output

A working design system in code: DTCG tokens, a deterministic build script, components on
headless primitives, a registry allowlist, CI enforcement, an interactive HTML preview for
approval, a published Storybook, and a generated skill file so the system can be used
without rediscovering it.

Not a strategy document.

## Sources

Grounded in primary sources: Nathan Curtis / EightShapes, Brad Frost, Radix, Material 3,
IBM Carbon, Shopify Polaris, Adobe Spectrum, GitHub Primer, Atlassian, Razorpay Blade,
W3C DTCG, WAI-ARIA APG, Figma's Code Connect evals, and published practitioner
postmortems. Contested points are flagged where experts disagree.

## Maintenance

`references/ai-agents.md` is the most time-sensitive file. Tooling claims and survey data
in it should be re-verified periodically; the underlying principles are more durable than
any specific tool named in it.
