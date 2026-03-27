---
name: skill-garden
description: Background knowledge for generating personalized .claude/ setups — how to read codebases for patterns, write effective skills, and keep them in sync.
user-invocable: false
---

# Skill Garden

Generator logic for the `compounding-skills` plugin. Loaded by `compounding-skills:setup`.

## Purpose

This skill teaches Claude how to:

1. **Read codebases for patterns** — extract real conventions from code, not hypothetical ones
2. **Write effective skills** — skills that use real examples, explain reasoning, and evolve over time
3. **Keep skills in sync** — detect when skills go stale and update them surgically

## Reference Documents

Load these as needed:

| File | When to use |
|------|-------------|
| `references/brownfield.md` | During `compounding-skills:setup` Phase 2A — step-by-step codebase analysis |
| `references/greenfield.md` | During `compounding-skills:setup` Phase 2B — preference interview questions and rationale |
| `references/workflow-templates.md` | During `compounding-skills:setup` Phase 5.1 — parametrized templates for each workflow skill |
| `references/skill-templates.md` | During `compounding-skills:setup` Phases 5.2–5.3 — skill output templates |
| `references/agent-templates.md` | During `compounding-skills:setup` Phases 5.5–5.6 — subagent definition templates |

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

**Testing description effectiveness:** After writing a description, mentally simulate these scenarios:

1. User asks Claude to "write a new controller" — does the expert skill description mention controllers?
2. User asks Claude to "fix this bug" — does the bug-hunter description mention debugging, error investigation?
3. User asks Claude to "add a background job" — does the expert skill mention the job framework?

If the answer is no for any common scenario, revise the description to include that context.

**Common undertriggering patterns to avoid:**
- Description only says what the skill *is* ("Rails conventions") instead of when to *use* it ("Use whenever writing, modifying, or reviewing any Ruby code")
- Description uses abstract language ("code quality") instead of concrete contexts ("services, controllers, models, policies, tests")
- Description is too short — better to use 80% of the 1024 character limit than 20%

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

## How to Write Effective Workflow Skills

The best workflow skills:

1. **Match the user's mental model** — if they think in terms of "tickets" not "plans", use that language
2. **Embed their tooling** — use their actual test and lint commands, not generic ones
3. **Reference their file structure** — tell them where to look in their actual project
4. **Are appropriately detailed** — a solo developer needs different verbosity than a team

See `references/workflow-templates.md` for parametrized templates.

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

## Skill Anatomy & Structure

When generating or updating any skill, follow this structure:

### Directory Layout

```
.claude/skills/{skill-name}/
  SKILL.md          # Core rules, brief examples, pointers to references
  README.md         # Optional: usage guide for user-invocable skills
  references/
    {topic-1}.md    # Detailed annotated examples
    {topic-2}.md    # More detailed examples
    scripts/        # Optional: bundled utility scripts
```

### Progressive Disclosure — Practical Word Counts

These are guidelines, not hard limits — go longer if the content earns its space:

1. **Description** (~50–100 words): What the skill does AND when to trigger it. Be pushy about triggering — "Always active when working in this repository" is better than "Rails patterns."

2. **SKILL.md body** (~300–800 words, under 500 lines): Core rules with brief inline examples (10–15 lines each). Every rule should explain why, not just what. Link to references for details.

3. **Reference files** (unlimited, but prefer 200–400 lines each): Full annotated examples with real file paths. Each example should demonstrate a pattern and explain the reasoning behind it.

### Writing Patterns

**Defining output formats:** When a skill needs the model to produce structured output, show the exact format with a real example rather than describing it abstractly.

**The examples pattern:** Show 1–2 real examples inline in SKILL.md, then point to references for more. Each example should demonstrate a different aspect of the pattern.

**The anti-pattern pattern:** When a rule is frequently violated, show both the wrong way and the right way, with reasoning for why the right way is better.

### Principle of Least Surprise

Generated skills should produce behavior that a developer familiar with the stack would expect. If a skill recommends an unusual approach, it must explain why the obvious approach doesn't work here.

## How to Think About Skill Improvements

When the compound skill proposes updates to skills, apply these filters:

### 1. Generalize, Don't Overfit

If a single work session revealed a pattern, that's interesting. If three sessions all needed the same pattern, that's a skill update. One data point is an observation; multiple data points are a convention. Don't add fiddly rules that only help the specific test cases you've seen.

### 2. Keep Skills Lean

Periodically read through generated skills and ask: "Did this section actually help in recent sessions?" If a rule hasn't influenced any generated code in the last several compound runs, consider removing it. Dead rules crowd out useful ones.

### 3. Explain the Why, Cut the Shouting

If you find yourself writing ALWAYS or NEVER in all caps, that's a signal the rule needs better reasoning, not more emphasis. Reframe: "NEVER use raw SQL" becomes "Use the ORM for queries because it handles escaping, logging, and connection pooling — raw SQL bypasses all three."

### 4. Bundle Repeated Work

If multiple work sessions result in writing similar utility scripts (API client generators, migration helpers, test data builders), that's a signal to bundle the script in the skill's `references/scripts/` directory so future sessions start with it already available. Only bundle scripts that are needed in 3+ work sessions — one-off scripts belong in the codebase, not the skill.
