# Skill Templates

Templates for generating `.claude/skills/` files. These skills are loaded automatically into Claude's context when relevant.

## Skill File Structure

All skill files follow this format:

```markdown
---
name: {skill-name}
description: {description — see "Writing Effective Descriptions" below}
disable-model-invocation: false
---

# {Skill Title}

{Brief explanation of what this skill covers and when it's relevant}

## {Section 1}

{Content with inline examples}

See [references/{topic}.md](references/{topic}.md) for detailed annotated examples.

## {Section 2}

{Content}
```

## Writing Effective Descriptions

The description field is the primary mechanism that determines whether Claude invokes a skill. Claude has a tendency to "undertrigger" — to not consult skills when they'd be useful. Combat this by making descriptions specific and slightly "pushy":

**Too vague (will undertrigger):**
> "Coding patterns for this Rails project"

**Good (specific contexts listed):**
> "Follow ProjectName conventions for all code generation — services, controllers, policies, models, views, testing. Always active when working in this repository. Use this whenever writing, modifying, or reviewing any Ruby code in this project."

Include both what the skill does AND specific contexts for when to use it. Keep under 1024 characters.

## Progressive Disclosure

Skills use a three-level loading system. Design generated skills to take advantage of this:

1. **Metadata** (~100 words) — Always in context. Name + description. Must be specific enough to trigger reliably.
2. **SKILL.md body** (< 500 lines ideal) — Loaded when triggered. Core rules, brief inline examples, pointers to references.
3. **Reference files** (unlimited) — Loaded as needed. Full annotated code examples, technique libraries.

**Key pattern:** SKILL.md contains the rules and brief (10-15 line) inline examples. Reference files contain the detailed (20-40 line) annotated examples with full context. This keeps the always-loaded content lean while keeping deep knowledge available.

---

## Template 1: Expert-Developer Skill

**Path:** `.claude/skills/expert-{stack}-developer/SKILL.md`
**Reference files:** `.claude/skills/expert-{stack}-developer/references/{layer}.md`

**Purpose:** An always-on rule set that enforces project-specific conventions for every code generation session. This is the most important generated skill — it shapes every line of code Claude writes in the project.

**Critical:** `disable-model-invocation: false` is required — this is what makes the skill always-active.

### SKILL.md Template

```markdown
---
name: expert-{stack}-developer
description: >
  Follow {project_name} conventions for all code generation — {layer-1}, {layer-2}, {layer-3}.
  Always active when working in this repository. Use this whenever writing, modifying, or
  reviewing any {language} code in this project, including new features, bug fixes, and refactors.
disable-model-invocation: false
---

# {Project Name} {Stack} Conventions

Follow these rules when generating or modifying any {stack} code in {project_name}.

## Architecture Overview

{Project name} is a **{framework version} {api_style}** {description}. Key architectural facts:

- **{key fact 1}**: {detail — e.g., dual user types, RBAC roles, namespace conventions}
- **{key fact 2}**: {detail}
- **Auth**: {authorization library and pattern}
- **Secrets/Env**: {how environment/secrets are managed}

## Code Style

- Use **{linter}** (`{lint_command}`). {Key style rules — e.g., no trailing commas, double-quote strings only when interpolation is needed.}
- **{key architectural principle}**: {e.g., "Thin controllers: Controllers handle HTTP concerns only. Business logic belongs in service objects because it's testable in isolation and reusable from background jobs."}

## {Layer 1 — e.g., Service Objects}

{Brief description of where this layer lives, its role, and *why* it exists in this architecture.}

- {Rule 1 — e.g., "Inherit from ApplicationService and implement #call — this gives you consistent error handling and logging across all services."}
- {Rule 2 — e.g., "Invoke with ClassName.call(params) — never ClassName.new(params).call — because the class method wraps the call in error handling."}
- {Rule 3 — e.g., "Wrap multi-step mutations in ActiveRecord::Base.transaction to ensure data consistency."}

```{language}
# Brief inline annotated example (~10-15 lines)
# Show the pattern in action with comments explaining *why*, not just *what*
{example code with inline comments}
```

See [references/{layer-1}.md](references/{layer-1}.md) for annotated examples.

## {Layer 2 — e.g., Controllers}

{Brief description and *why* this layer is structured this way.}

- {Rule 1}
- {Rule 2}
- {Rule 3}

```{language}
# Brief inline example
{example code}
```

See [references/{layer-2}.md](references/{layer-2}.md) for annotated examples.

## Error Handling

| Error Class | HTTP Status | Use When |
|---|---|---|
| `{ErrorClass1}` | {4xx} | {description} |
| `{ErrorClass2}` | {4xx} | {description} |
| `{ErrorClass3}` | {4xx} | {description} |
```

### Reference File Template

**Path:** `.claude/skills/expert-{stack}-developer/references/{layer}.md`

```markdown
# {Layer} — {Stack} Conventions

## {Pattern Name — e.g., Standard Service Object}

*Source: `{actual_file_path}:{line_range}`*
(Brownfield: real file. Greenfield: *Template — evolve via `{command_prefix}:compound`*)

```{language}
# Key rules demonstrated in this example:
# 1. {what rule this line shows — and why it matters}
# 2. {what this block demonstrates — and the reasoning behind it}
{annotated code excerpt — 20-40 lines max}
# ↑ {annotation explaining a key pattern and its purpose}
```

### Key rules for this layer:

- {Rule 1 — matches annotation 1 above, with reasoning}
- {Rule 2 — matches annotation 2 above, with reasoning}
- {Rule 3}

## {Pattern Name 2 — optional second example for this layer}

*Source: `{actual_file_path}:{line_range}`*

```{language}
{second annotated example}
```
```

**Brownfield workflow for each reference file:**
1. `Glob` the layer's directory (e.g., `app/services/**/*.rb`) to find candidates
2. `Grep` for base class or key patterns to find the most representative file
3. `Read` the file, select the most instructive 20-40 lines
4. Annotate with inline comments explaining *why*, not just *what* — reasoning helps the model apply patterns correctly in novel situations
5. Write — do NOT copy entire files

---

## Template 2: New Pattern Skill (from {command_prefix}:compound)

**Path:** `.claude/skills/{pattern-name}/SKILL.md`

Used when `{command_prefix}:compound` identifies a significant new pattern that warrants its own skill.

```markdown
---
name: {pattern-name}
description: >
  {One-line description — when and why to use this pattern}
  Use this whenever {specific trigger contexts — be pushy to combat undertriggering}.
disable-model-invocation: false
---

# {Pattern Name}

*Created {date} — emerged from {brief context of what triggered creation}*

## When to Use

{1-2 sentences describing the situation this pattern addresses and *why* this approach was chosen.}

## The Pattern

```{language}
{real code example from the codebase, with file path reference}
```

*Source: `{actual_file_path}:{line_number}`*

## Why This Approach

{Explanation of why this pattern was chosen over alternatives. This reasoning helps the model make good judgment calls in edge cases.}

## What to Avoid

```{language}
{anti-pattern or alternative to avoid}
```

*Reason: {brief explanation of the problem this causes}*

## Evolution Log

- {date}: Pattern created from {context}
```

---

## Adding Examples via `/{command_prefix}:compound`

When `{command_prefix}:compound` adds an example to an existing skill, use this format:

```markdown
### {Pattern Name} — {Specific Variation}

*Added {date} — from {brief context}*

```{language}
# {file_path}:{line_number}
{real code excerpt}
```

*Why this works: {1-2 sentence explanation of the reasoning behind this approach}*
```

Insert after the most relevant existing section. Do not append to the end of the file — place examples near related content.

**When to remove:** If a new example makes an existing generic example redundant, replace the generic one. If a section consistently isn't being followed (visible in code review findings), investigate whether the rule is wrong rather than adding more emphasis. Skills that explain *why* get followed; skills that shout ALWAYS get ignored.

---

## Bundling Utility Scripts

When a pattern involves running a script (data migration, API client generation, test data seeding), bundle it in the skill rather than making every work session reinvent it.

**Path:** `.claude/skills/{skill-name}/references/scripts/{script-name}.{ext}`

**Reference from SKILL.md:**
> "For {task}, use the bundled script at `references/scripts/{script-name}.{ext}` — it handles {what it handles}."

**When to bundle:** Only bundle scripts that are needed in 3+ work sessions. One-off scripts belong in the codebase, not the skill. The signal is clear when multiple independent work sessions all end up writing similar helper scripts — that repeated work should be captured once and reused.

**When NOT to bundle:** Don't bundle application-specific scripts that belong in the project's own `scripts/` or `bin/` directory. Skill scripts should be generic enough to help across many prompts.
