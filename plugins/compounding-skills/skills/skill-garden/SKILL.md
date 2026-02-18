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
2. **Write effective skills** — skills that use real examples and evolve over time
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

### 1. Real Examples Beat Generic Advice

A skill that says "use service objects for complex business logic" is less useful than one that says:
> "In this codebase, complex business logic lives in `app/services/` — see `app/services/subscription_renewal_service.rb:23` for a canonical example."

Always prefer real file paths and real code over generic descriptions.

### 2. Skills Should Be Surgically Updatable

Write skills so individual sections can be updated without rewriting the whole file. Use clear section headers. Each pattern should be a standalone unit.

### 3. Less Is More

A skill with 3 well-chosen examples is more useful than one with 20 mediocre ones. When `{command_prefix}:compound` adds examples, prefer replacing a weak generic example with a real one over appending endlessly.

### 4. Skills Reflect Current Reality

If the codebase refactored away from a pattern, remove it from the skill. Outdated guidance is worse than no guidance — it actively misleads.

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
2. **Show real examples** — "❌ Over-engineered (like the old user_presenter.rb) → ✅ Simpler (like the refactored version)"
3. **List stack-specific rules** — Rails: thin controllers; TypeScript: avoid wrapper types; etc.
4. **State preferences explicitly** — "This codebase prefers explicit error handling over implicit"

The agent should feel like it was written by someone who has read all the code, not copied from a style guide.
