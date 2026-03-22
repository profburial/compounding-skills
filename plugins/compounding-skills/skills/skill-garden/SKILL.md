---
name: skill-garden
description: Background knowledge for generating personalized .claude/ setups — how to read codebases for patterns, write effective skills, and keep them in sync.
user-invocable: false
---

# Skill Garden

Generator logic for the `compounding-skills` plugin. Loaded by `compounding-skills-setup`.

## Purpose

This skill teaches Claude how to:

1. **Read codebases for patterns** — extract real conventions from code, not hypothetical ones
2. **Write effective skills** — skills that use real examples, explain reasoning, and evolve over time
3. **Keep skills in sync** — detect when skills go stale and update them surgically

## Reference Documents

Load these as needed:

| File | When to use |
|------|-------------|
| `references/brownfield.md` | During `compounding-skills-setup` Phase 2A — step-by-step codebase analysis |
| `references/greenfield.md` | During `compounding-skills-setup` Phase 2B — preference interview questions and rationale |
| `references/command-templates.md` | During `compounding-skills-setup` Phase 5.1 — parametrized templates for each workflow command |
| `references/skill-templates.md` | During `compounding-skills-setup` Phases 5.2–5.3 — skill output templates |
| `references/agent-templates.md` | During `compounding-skills-setup` Phases 5.5–5.6 — subagent definition templates |

## Core Principles

### 1. Explain the Why, Not Just the What

Today's LLMs are smart. They have good theory of mind and when given reasoning can go beyond rote instructions. A skill that says "Always use service objects" is less useful than one that says:

> "Business logic lives in `app/services/` because controllers should only handle HTTP concerns — mixing business logic into controllers makes it untestable outside a request context and leads to duplication when the same logic is needed from a background job."

When writing skills, explain *why* a convention exists. This lets the model make good judgment calls in edge cases rather than blindly following rules. If you find yourself writing ALWAYS or NEVER in all caps, reframe it as reasoning the model can internalize.

### 2. Real Examples Beat Generic Advice

A skill that says "use service objects for complex business logic" is less useful than one that says:
> "In this codebase, complex business logic lives in `app/services/` — see `app/services/subscription_renewal_service.rb:23` for a canonical example."

Always prefer real file paths and real code over generic descriptions. Real examples ground the model in how *this* project actually works.

### 3. Progressive Disclosure

Skills use a three-level loading system. Understanding this helps write skills that are effective without wasting context window space:

1. **Metadata** (name + description) — Always in context. ~100 words max. This is the primary trigger mechanism — include both what the skill does AND when to use it.
2. **SKILL.md body** — Loaded when the skill triggers. Keep under 500 lines. Contains the core rules and brief inline examples.
3. **Bundled references** — Loaded as needed. Unlimited size. Contains detailed annotated examples, technique libraries, etc.

Put the most important information in the SKILL.md body. Detailed examples and reference material go in `references/` files with clear pointers about when to read them.

### 4. Skills Should Be Surgically Updatable

Write skills so individual sections can be updated without rewriting the whole file. Use clear section headers. Each pattern should be a standalone unit. This matters because `{command_prefix}:compound` needs to insert new examples near related content.

### 5. Less Is More

A skill with 3 well-chosen examples is more useful than one with 20 mediocre ones. Every line in a skill costs context window space — unused instructions actively hurt by crowding out useful ones.

When `{command_prefix}:compound` adds examples, prefer replacing a weak generic example with a real one over appending endlessly. If a section isn't contributing to better outcomes, remove it.

### 6. Skills Reflect Current Reality

If the codebase refactored away from a pattern, remove it from the skill. Outdated guidance is worse than no guidance — it actively misleads.

### 7. Write Descriptions That Trigger Reliably

Claude has a tendency to "undertrigger" skills — to not use them when they'd be useful. Combat this by making descriptions a little "pushy." Instead of:

> "Coding patterns for this Rails project"

Write:

> "Follow ProjectName conventions for all code generation — services, controllers, policies, models. Always active when working in this repository. Use this whenever writing, modifying, or reviewing any code."

Include specific contexts for when to use the skill, not just what it does.

## How to Analyze Code for Patterns

When reading a codebase (brownfield analysis), look for signals:

### Structural Signals
- Directory layout: MVC vs. feature-folder vs. flat?
- File naming: `UserService` vs. `user_service.rb` vs. `user-service.ts`?
- Where do edge cases live? Are there `concerns/`, `mixins/`, `lib/` directories?

### Complexity Signals
- Function length: 5-line methods vs. 50-line methods — which is more common?
- Abstraction depth: how many layers deep does a typical call stack go?
- Comment style: verbose explanations vs. self-documenting code?
- Explicit vs. implicit: `user.save!` vs. `begin; user.save!; rescue => e; ...`?

### Convention Signals
- Consistent naming? e.g., all services end in `Service`, all workers in `Worker`
- Consistent error handling? e.g., always return result objects, never raise
- Consistent data access? e.g., always use scopes, never `.where` in controllers

### Why Signals (Most Important)

Don't just document *what* the patterns are — understand *why* they exist:
- Is the project structured this way for testability? Performance? Team conventions?
- Were certain patterns chosen because of specific constraints (compliance, scale, team size)?
- What pain did the team hit that led to this convention?

Capture the reasoning when it's apparent from code comments, commit messages, or the structure itself. Skills that explain *why* produce better results than skills that just list *what*.

## How to Write Effective Workflow Commands

The best workflow commands:

1. **Match the user's mental model** — if they think in terms of "tickets" not "plans", use that language
2. **Embed their tooling** — use their actual test and lint commands, not generic ones
3. **Reference their file structure** — tell them where to look in their actual project
4. **Are appropriately detailed** — a solo developer needs different verbosity than a team

See `references/command-templates.md` for parametrized templates.

## How the Code-Simplifier Agent Works

The `code-simplifier` agent is the most personalized artifact. It should:

1. **Open with the project's complexity bar** — "In this codebase, the right level of complexity is X"
2. **Show real examples** — "Over-engineered (like the old user_presenter.rb) vs. Simpler (like the refactored version)"
3. **List stack-specific rules with reasoning** — not just "thin controllers" but "thin controllers because business logic in controllers can't be tested outside a request context and gets duplicated when needed from jobs"
4. **State preferences explicitly** — "This codebase prefers explicit error handling over implicit"

The agent should feel like it was written by someone who has read all the code, not copied from a style guide.

## Writing Style for Generated Skills

Use the imperative form in instructions. Prefer explaining reasoning over heavy-handed directives:

**Weaker:** "ALWAYS use guard clauses. NEVER nest more than 2 levels."

**Stronger:** "Prefer guard clauses and early returns — they reduce nesting depth and make the happy path easier to follow. Deep nesting forces readers to mentally track multiple conditions simultaneously."

The model will follow well-reasoned guidance more reliably than rigid rules, and it will make better judgment calls in edge cases the rules didn't anticipate.
