# Contributing to the MLPerf Endpoints submitter documentation

This site is a **guided path over** the authoritative sources, not a copy of them. Most review
comments on a docs PR come down to one of the rules below, so they are worth reading once.

## The five contracts

1. **Policy is never restated.** Summarise a requirement in submitter language and deep-link to the
   clause that says it. Every rules page carries the precedence notice: if this site and
   `mlcommons/endpoints_policies` disagree, the policy repository wins.
2. **One home per fact.** Package structure lives in `reference/package-layout.md`. Field tables
   live in `reference/`. If you find yourself explaining a `point.yaml` field on a workflow page,
   link instead.
3. **Link direction is fixed.** Workflow → Reference (detail), Workflow → Rules (constraints),
   Workflow → Help (failure), Workflow → Understand (orientation). Reference pages do not chain to
   each other.
4. **Genre per section.** Workflow pages are procedures: no exhaustive option tables, no theory.
   Understand pages are explanation: no commands. Reference pages are description: no narrative.
   A paragraph that feels awkward is usually on the wrong kind of page.
5. **Nothing is invented.** Every command, flag, field name and numeric threshold must trace to a
   source in `_sources/` or to the upstream repositories. If you cannot cite it, it belongs in
   `help/open-questions.md` as a question, not in a page as a fact.

## `Last verified`

Every rules and reference page ends with a `Last verified against:` line naming the source and its
ref. The upstream rules are a live development branch; undated derived documentation of a moving
draft is worse than none. When you touch a page, re-check its claims and update the date — or
don't touch it.

## Page templates

**Workflow step:** goal line → `!!! note "Before you begin"` → *What you'll do* → *Steps* → rule
callouts → **Verify** → *Next*. The **Verify** section is mandatory and must contain a command or
an observable outcome, not a reassurance.

**Reference page:** one-line scope + link to the conceptual explanation → tables → example →
caveats in a collapsible block.

## Admonition budget

- `danger` — **only** for rules whose failure action in Endpoints Rules §9.1 is *Reject submission*.
- `warning` — other binding constraints.
- `note` / `tip` — everything else.

If everything is a warning, nothing is.

## Build

```bash
pip install mkdocs-material
mkdocs serve          # local preview
mkdocs build --strict # what CI runs; a broken internal link fails the build
```

## FAQ rule

An FAQ entry that keeps getting asked is a **documentation bug**. Fix the workflow page that should
have answered it, then delete the entry. Without this rule the FAQ becomes the real documentation.
