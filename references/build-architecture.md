# Build architecture: file shape, layer boundaries, and the verify gate

How the system is arranged on disk, and which structural rules CI enforces. Token
architecture is in `tokens.md`, what to build is in `components.md`, agent-facing delivery
is in `ai-agents.md` and `delivery.md`. This file is about the skeleton those hang on.

Examples use React and TypeScript because they need to be concrete. The principles are
stack-neutral.

---

## The golden path: one uniform file shape

Every component, at every layer, has the identical file set:

```
components/ui/button/
├── button.tsx          the implementation
├── button.spec.md      the decisions the API cannot express
├── button.test.tsx     one behavioral check: states and a11y
└── index.ts            re-export
```

**Uniformity is the point, and it matters more with agents than it ever did with humans.**
An agent learns the pattern once and applies it to component 40 exactly as it did to
component 1. A human eventually notices an inconsistent layout and adapts. An agent
propagates it. When every component is shaped identically, "where does the spec go" is
never a decision, and a missing file is mechanically detectable.

The variation to resist: letting complex components sprout extra files ad hoc. If a
component genuinely needs sub-parts, use a `parts/` subdirectory inside the same shape
rather than inventing a new shape.

### The spec file is the load-bearing part

`component.tsx` says what the component does. `component.spec.md` says **why it is shaped
that way**, which is exactly what an agent cannot infer from an API and what a new hire
asks in review.

Template:

```markdown
# Button

> Primary action trigger. Prefer over a link when the action changes state.

## When to use / not use
- USE for actions that change state (submit, delete, apply).
- DON'T use for navigation, use Link. DON'T put more than one primary per view.

## Variants (must match the variant function in button.tsx)
- variant: primary | secondary | tertiary | destructive
- size:    sm | md | lg

## Tokens (never hard-code)
- background: action.primary.rest / .hover / .pressed
- text:       content.on-primary
- padding:    space.12 / space.16
- radius:     radius.full

## States (all required)
rest · hover · pressed · focus-visible · disabled · loading

## Accessibility
- Focus ring always visible, never removed.
- Disabled uses aria-disabled, state never by color alone.
- Target minimum 44x44.
```

Keep specs short. A spec nobody reads because it is three pages long is worse than a
half-page one that gets maintained.

---

## Layer boundaries, enforced

Layers form a directed acyclic graph, and the direction is one-way:

```
tokens -> icons -> atoms -> molecules -> organisms
                                  utils (leaf, importable by anyone)
```

An atom may never import a molecule. A molecule may never import an organism. The layer
names are the same ones used in the canonical inventory (`components.md`) and in the HTML
preview sections (`delivery.md`), so one vocabulary runs from preview to shipped code.

**Enforce this with a script that fails CI, not a code review convention.** Scan imports,
map each file and each import to a layer rank, and fail when a lower rank imports a higher
one.

Why this matters more when agents write the code: an agent has no instinct for
architectural direction. It will happily import a `Card` into a `Button` to reuse a border
style, produce code that compiles and renders perfectly, and create a cycle that only
surfaces months later as a circular dependency or an impossible refactor. Nothing in
typecheck, lint, or tests catches it. This is the same class of gap as a registry allowlist:
a cross-artifact rule that no single-file check can see.

Whether you use real packages (a monorepo with workspaces) or just directories with a
lint boundary rule is a scale decision. The enforcement matters; the packaging does not.

---

## The variant contract

Declare variants in **one place**, and treat that declaration as the component's public
API. A variant function (CVA is the common implementation, but the idea is not tied to it)
gives you a single readable block that both a human and a model can read as the contract.

The rule that follows: **no ad-hoc `className` branching for visual variants.** If a
consumer needs a different look, it is a variant that does not exist yet, not a prop to
override. This is the same line drawn in `components.md` on escape hatches: content
composition is free, visual override is a system gap.

Keep the variant names identical to the Figma or design-source variant property names,
verbatim, including casing. That 1:1 mapping is what removes the translation layer for both
engineers and codegen.

---

## One ordered verify command

Wire every gate into a single command (`verify`, `check`, whatever you name it) that runs
them in a deliberate order and fails the build on the first non-zero exit.

| # | Gate | Catches |
|---|---|---|
| 1 | build | the whole graph compiles |
| 2 | token drift | committed generated output differs from a clean rebuild |
| 3 | hard-coded value audit | hex, rgb, arbitrary spacing in source |
| 4 | boundary audit | a layer importing upward |
| 5 | spec-existence audit | a shipped component with no spec file |
| 6 | registry check | an import that resolves to nothing in the allowlist |
| 7 | semantic heading audit | a styled div standing in for a real heading element |
| 8 | anchor integrity audit | an internal link pointing at an id that does not exist |
| 9 | types | invalid token names, invalid props |
| 10 | tests | behavior and a11y |

**The ordering is not cosmetic.** A later gate's result is meaningless if an earlier one is
broken: there is no point auditing token usage in a tree that does not compile, and no
point running tests against stale generated CSS. Fail fast, in dependency order.

One command matters because it is the thing you tell an agent to run, and the thing CI
runs. Two commands means one of them gets skipped.

---

## The token drift gate

Committed generated output must equal a clean rebuild. Give the token build a `--check`
flag that rebuilds into a temp location and diffs against what is committed.

This catches the two most common silent failures: someone hand-edited generated CSS, or
someone changed a token and never re-ran the build. Both leave a repo where the source of
truth and the shipped values disagree, and nothing else notices.

The same principle generalizes to any generated artifact: the registry, the generated skill
file, a docs index. **If it is generated, CI should be able to prove it is current.**

---

## The spec-existence audit

A script that fails the build when a shipped component has no spec file.

Worth calling out separately because it is counter-intuitive: the code compiles, renders,
passes tests, and ships perfectly fine without a spec. There is no natural pressure to
write one. So the only thing that keeps specs from rotting into optional is a gate that
treats a missing one as a build failure.

Same family as the registry allowlist and the boundary audit: cross-artifact checks that
catch what no single-artifact gate can see.

---

## The semantic heading audit

**Every element that presents as a heading must be a real heading element.** If it carries
a heading-ish class, sits in a heading slot, or reads as a section title, it is `h1`
through `h6` or it fails. Levels may not skip, either: an `h4` directly under an `h2` is a
broken outline even though both are real elements.

This is worth a gate because it is invisible to everything else. A styled div renders
identically, compiles, passes tests, and survives a visual diff without a pixel of
difference. What it destroys is the document outline: screen-reader users navigate by
heading as a primary mode and see almost nothing, and any tooling that walks the heading
tree (a generated table of contents, an outline view, a docs indexer) sees the same
nothing. In one reviewed generated preview, over a hundred styled divs were acting as
component headings against roughly thirty real heading elements.

Agents produce this constantly, and the reason is structural rather than careless. Agent
output is styling-led: it reaches for a div plus a class because that is where the visual
intent lives, and the semantic element is an afterthought that nothing punishes.

Implementation: parse the rendered output or the source, collect every element carrying a
heading-ish class or `role="heading"`, and fail on any that is not `h1` through `h6`. Then
walk the resulting heading tree in document order and fail on a level skip.

The one legitimate exception is `role="heading"` with an explicit `aria-level`, which is
the correct answer when a real heading element genuinely cannot be used. Allow that pairing
explicitly in the gate rather than blanket-failing, and require `aria-level` to be present
so the exception still carries its outline position.

---

## The anchor integrity audit

**Every internal `href="#id"` must resolve to an element carrying that id in the same
document.** No exceptions worth carving out.

A dead in-page link is normally a small annoyance. It stops being small when the
navigation doubles as a completeness checklist, which is exactly the arrangement the
preview uses (`delivery.md`). An entry that does not resolve is the artifact asserting that
a component exists when it does not. A checklist that lies is worse than no checklist,
because it is trusted. In one real case a generated navigation carried eight links pointing
at ids that were never emitted, and the reviewer had no way to tell without clicking every
one.

Implementation is about ten lines: collect every `href` beginning with `#`, collect every
`id` in the document, and fail on the set difference. Report the dead anchors by name so
the fix is obvious.

Generalize it past the preview. Any generated index, nav, table of contents, or docs
sidebar deserves the same check, and so does any cross-file link scheme if your docs are
multi-page. It is the natural companion to the spec-existence audit: both assert that a
thing being claimed actually exists, one for files and one for anchors.

The registry allowlist, the boundary audit, the spec-existence audit, and both gates above
share one property worth naming. **They are cross-artifact or structural checks that pass
every single-file gate**, because the code compiles, renders, and looks correct. That is
precisely the category of defect agents produce most reliably, and the only defense is a
gate that compares one artifact against another rather than checking either alone.

---

## A greppable escape hatch

The hard-coded-value audit should support an inline marker that exempts one line:

```tsx
// tokens-ok: third-party embed requires a literal hex
backgroundColor: '#1877F2',
```

This is deliberately better than either extreme. **No escape** means teams eventually
disable the rule wholesale, or fork the component. **A silent escape** (a config exclusion,
a disabled lint rule) means the exception is invisible and accumulates.

A marker in the source is explicit, requires a reason, shows up in review, and is one grep
away from an audit. Count them periodically. A rising count is a signal your token set is
missing something real.

---

## On discovery files, honestly

An independent architecture document reviewed for this skill reached the same conclusion
`ai-agents.md` already holds about `llms.txt`: worth doing because it is cheap, not because
it works, with the observation that the large majority of published `llms.txt` files are
never actually read. Two sources arriving at that independently is worth one line of
confidence. Publish it if you like; do not count on it. A registry or MCP the agent can
query is the mechanism with evidence behind it.

---

## Contested

- **Uniform file shape has real ceremony cost.** Four files per component is heavy for a
  system with 12 components and a single maintainer. Below roughly 20 components, folding
  the spec into a docs comment or a Storybook docs page is defensible. Above that, the
  uniformity pays for itself. Know which side of the line you are on rather than adopting
  the ceremony reflexively.
- **Packages versus directories.** Real workspace packages give you genuine enforcement and
  independent versioning at the cost of build complexity. Directories plus a lint rule get
  you most of the enforcement for none of the setup. Start with directories unless you are
  publishing the system for external consumption.
- **How strict the boundary rule should be.** A hard failure is correct for atoms and
  molecules. Organisms importing from a shared utils leaf is fine. Be precise about the
  exceptions in the rule's config rather than loosening it globally.
