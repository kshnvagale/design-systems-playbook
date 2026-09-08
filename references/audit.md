# Auditing and reviewing an existing design system

The rest of this skill assumes you are building. This file is for when you are assessing
something that already exists: a health check, a due-diligence pass, or the first step
before a migration. Read this before you run any part of the greenfield discovery batch.

## Ask for the repository first. A Figma file is not sufficient.

This is not negotiable, and it gates everything else in this file.

An audit needs to inspect what actually ships: the token source, the component
implementations, the lint and CI configuration, the registry if one exists, and how the
system is really consumed across product code. A Figma library shows intent. Code shows
reality, and the gap between the two is usually the single most valuable finding an audit
produces. You cannot check token coverage, layer boundaries, spec existence, or drift from
a design file alone.

Ask for, in order of value:

1. **The design system's own repository.** Non-negotiable for anything beyond a surface
   opinion.
2. **At least one consuming product repository**, so you can measure real usage and
   coverage rather than what the system's own docs claim.
3. **A published Storybook or docs URL**, if one exists.
4. **The Figma library**, as a supplement for comparing design intent against shipped
   code, not as a substitute for the repo.

**If they cannot provide a repository, do not refuse outright.** Say plainly what cannot
be assessed without it: token coverage, enforcement, drift, real adoption, layer
integrity. Offer a reduced scope based on whatever is available. A deployed URL alone
still lets you inspect computed styles for token usage, walk the DOM for coverage, and
run automated accessibility checks. If there is genuinely nothing to inspect, say so
directly and pivot to the greenfield path in `SKILL.md`, stating why you are pivoting
rather than silently switching tracks.

## An audit is a valid terminal deliverable

The output of this file is a report. It is not automatically a prelude to a rebuild.
Do not push toward a rewrite unless the evidence in your findings actually supports one.
Most audited systems need targeted fixes, not a restart; see the rebuild signals near the
end of this file before recommending one.

## Scope discovery down for an audit

The full discovery batch in `discovery.md` is built for a greenfield build and asks the
wrong questions here. Keep only:

- Product type and platforms
- Who maintains the system today, and at what real capacity
- The accessibility bar that applies
- Who and what writes the UI code now (humans, agents, or both)
- Fixed constraints you must audit within, not around

Skip the rest. Visual-direction questions, brand color, and moodboard references are
irrelevant to assessing what already exists.

## The audit procedure

Work through these layers in order. Each reuses a rubric that already exists elsewhere
in this skill rather than inventing a new one.

### Foundations

- Token tiers: does a real primitive/semantic split exist, or do components consume raw
  values directly? See `tokens.md` for the tier model and its failure modes.
- Do primitives leak into components, bypassing the semantic layer entirely?
- Palette size: is every hue justified by a semantic token, or is there unused surplus?
  See `color.md` for the restraint rule and the common failure of generating hues nobody
  named a job for.
- Contrast across every real token pairing, in every theme the system ships. Do not
  spot-check; enumerate pairings.
- Spacing, radius, and type scale coarseness: are there off-scale values already in use
  that the token set never accounted for?

### Naming

Apply the naming smells list in `naming.md` directly against the existing inventory.
Specifically check for the ARIA-alignment failures that recur across real systems:
components literally named `Dropdown` instead of `Listbox`/`Menu`/`Combobox`, `Modal`
used where `Dialog` is the correct ARIA-aligned name, and a `Chip`/`Tag`/`Pill` cluster
that should be one component.

### Components

- Coverage against the canonical inventory in `components.md`. What exists, what is
  missing, what was built that never should have been (product-specific one-offs living
  in the shared system).
- API consistency across components: same concept named differently in different
  places, inconsistent prop shapes for the same kind of decision.
- Variant explosion: components with props that overlap or that only make sense in
  combination.
- Missing states: hover, focus, disabled, loading, error, present in some components and
  silently absent in others.
- Escape-hatch usage: how often is `className`/`style` overriding visual props instead of
  a variant existing for the need. A high count is a proxy for missing variants.

### Architecture

Check file-shape uniformity and layer boundaries against `build-architecture.md`. Does
every component follow the same file shape? Does an atom ever import a molecule? Is there
a real dependency direction, or has it eroded over time?

### Enforcement

List every CI gate that exists today, and every one this skill recommends that is
absent. Then ask the harder question for each gate that does exist: **has it ever been
observed to fail?** A rule that has never fired once in the project's history is
indistinguishable from a no-op, and you should treat it as broken until proven otherwise.
See `evaluation.md` for why this matters and how to fixture-test a gate.

### Adoption

Measure coverage the way `evaluation.md` recommends: percentage of real UI actually
composed from the design system, not import counts. Check whether component and prop
usage is instrumented at all. Look for components with zero real usage, which are
maintenance cost with no return.

### Governance

Assess the team model against what is actually staffed, not what is documented.
Check contribution process, release cadence, and the health signals in `governance.md`:
is this system shipping regularly, or has it gone quiet while still being called
"maintained."

## The audit report

Produce a single report with these sections:

**Scope.** What was inspected: which repositories, which URLs, what was and was not
available, and what that means for confidence in the findings.

**Findings table.** One row per finding: the finding itself, severity (High / Medium /
Low), evidence (file and line, or a reproducible observation), and a suggested fix.
Do not report a finding without evidence a reader could go verify themselves.

**What could not be assessed, and why.** State this explicitly rather than silently
omitting a section because the input was incomplete.

**What the system does well.** Same evidence standard as the findings table. An audit
that is purely negative is either incomplete or biased; real systems almost always have
things worth keeping.

**Remediation order.** A prioritized sequence, not just a severity-sorted list. Some
Medium findings block other fixes and should move ahead of independent High findings.

## Migration and deprecation

For any audit that leads to changes, follow this before touching a single value:

1. **Inventory what exists before changing anything.** You cannot measure a successful
   migration without a starting count.
2. **Map old values to the nearest new primitive or semantic token.** Do this
   deliberately, not by pattern-matching hex values; two visually similar colors may
   carry different meanings.
3. **Alias windows.** Keep a deprecated token resolving to its old value, pointed at the
   new one under the hood, for at least one release. Never remove a token the same
   release you rename it.
4. **Codemods for mechanical renames.** Anything that is a pure find-and-replace across
   the codebase should be scripted, not done by hand file by file.
5. **Instrument who still consumes a deprecated token or component before removing it.**
   Removing something still in use is a regression, not a cleanup.
6. **The legacy folder pattern.** Components not yet migrated live in an explicitly
   labeled legacy location, so nobody mistakes them for the current standard while the
   migration is in progress.
7. **Migrate by traffic, not by page count.** Prioritize high-traffic surfaces first. A
   migration that hits every low-traffic admin page before the homepage has its
   priorities backwards.

## When an audit should become a rebuild

Most of the time it should not. Rebuilds are proposed far more often than they are
warranted, and a rebuild recommendation should be treated with real suspicion, including
your own.

**Real signals that a rebuild is justified:**

- No semantic layer exists at all, so nothing in the system can be themed without
  touching every component.
- Naming has no recoverable taxonomy: names carry no consistent logic to reverse-engineer
  a migration path from.
- No enforcement exists, and drift has progressed to the point where migrating the
  existing surface costs more than starting over.

**Counter-signals, worth weighing seriously before recommending a rebuild:**

- The problems found are concentrated in a few layers (naming, or enforcement, or
  coverage) rather than pervasive. Targeted fixes address concentrated problems more
  cheaply than a rebuild does.
- The system has real, working adoption today. A rebuild resets that adoption to zero
  and the migration itself may cost more than the technical debt it removes.
- The team lacks the capacity to execute a rebuild well, which is a governance problem
  (see `governance.md`) independent of the system's technical state, and a rebuild will
  not fix it.

## Contested

- **How much weight to give a system's own documentation versus its actual code.**
  Documentation drifts from reality faster than code does. When they disagree, trust
  what ships, and note the documentation gap as its own finding.
- **Whether a system with zero failed-gate history should be treated as clean or as
  unverified.** This file takes the position that it is unverified until proven
  otherwise, which is stricter than some teams would apply to their own systems.
