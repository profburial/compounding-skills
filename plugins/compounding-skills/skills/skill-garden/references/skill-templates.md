# Skill Templates

Templates for generating `.claude/skills/` files. These skills are loaded automatically into Claude's context when relevant.

## Skill File Structure

All skill files follow this format:

```markdown
---
name: {skill-name}
description: {one-line description of what this skill provides}
user-invocable: false
---

# {Skill Title}

{Brief explanation of what this skill covers and when it's relevant}

## {Section 1}

{Content}

## {Section 2}

{Content}
```

---

## Template 1: Stack-Patterns Skill

**Path:** `.claude/skills/{stack}-patterns/SKILL.md`

**Purpose:** Documents the actual coding patterns, conventions, and examples from this specific codebase.

### Brownfield Version

```markdown
---
name: {stack}-patterns
description: Real coding patterns and conventions extracted from this {stack} codebase
user-invocable: false
---

# {Stack} Patterns

Patterns observed in this codebase. All examples are from real files.

## Key File Locations

| Type | Where they live | Example |
|------|-----------------|---------|
| {type 1} | `{actual_path}/` | `{actual_file_path}` |
| {type 2} | `{actual_path}/` | `{actual_file_path}` |

## Naming Conventions

- Files: `{observed_naming_pattern}` (e.g., `user_subscription_service.rb`, `UserSubscriptionService.ts`)
- Classes: `{class_naming}` (e.g., `UserSubscriptionService`)
- Methods: `{method_naming}` (e.g., `process_renewal`, `processRenewal`)

## Core Patterns

### {Pattern Name}

*Observed in `{actual_file_path}:{line_number}`*

```{language}
{real code excerpt — actual content from the file, not invented}
```

This pattern is used for {explanation of when this applies}.

### {Pattern Name 2}

*Observed in `{actual_file_path}:{line_number}`*

```{language}
{real code excerpt}
```

## What NOT to Do

Based on what's NOT present in this codebase:

- Avoid: {anti-pattern} — this codebase {never uses / has moved away from} this
- Prefer: {preferred pattern} instead
```

### Greenfield Version

```markdown
---
name: {stack}-patterns
description: Preferred coding patterns for this {stack} project (to be evolved via `/{command_prefix}:compound`)
user-invocable: false
---

# {Stack} Patterns

> **Note:** This skill was bootstrapped from stated preferences, not observed code.
> Run `{command_prefix}:compound` after each feature to replace these defaults with real examples
> from your actual codebase.

## File Organization

{Based on Q2 answer: by-type, by-feature, framework-default, or flat}

Example layout:
```
{appropriate directory tree for chosen organization style}
```

## Naming Conventions

{Stack-appropriate naming conventions}

## Core Patterns

### Typical Service / Business Logic

```{language}
{stack-appropriate idiomatic example — this is a generic starter}
```

### Error Handling

```{language}
{stack-appropriate error handling pattern}
```

## Testing Patterns

{Based on Q3 answer}

```{language}
{typical test structure for chosen approach}
```
```

---

## Template 2: New Pattern Skill (from {command_prefix}:compound)

**Path:** `.claude/skills/{pattern-name}/SKILL.md`

Used when `{command_prefix}:compound` identifies a significant new pattern that warrants its own skill.

```markdown
---
name: {pattern-name}
description: {One-line description — when and why to use this pattern}
user-invocable: false
---

# {Pattern Name}

*Created {date} — emerged from {brief context of what triggered creation}*

## When to Use

{1-2 sentences describing the situation this pattern addresses}

## The Pattern

```{language}
{real code example from the codebase, with file path reference}
```

*Source: `{actual_file_path}:{line_number}`*

## Why This Approach

{Explanation of why this pattern was chosen over alternatives}

## What to Avoid

```{language}
{anti-pattern or alternative to avoid}
```

*Reason: {brief explanation}*

## Evolution Log

- {date}: Pattern created from {context}
```

---

---

## Template 4: Expert-Developer Skill

**Path:** `.claude/skills/expert-{stack}-developer/SKILL.md`
**Reference files:** `.claude/skills/expert-{stack}-developer/references/{layer}.md`

**Purpose:** An always-on rule set that enforces project-specific conventions for every code generation session. This replaces the generic stack-patterns skill with one that contains real architectural knowledge.

**Critical:** `disable-model-invocation: false` is required — this is what makes the skill always-active.

### SKILL.md Template

```markdown
---
name: expert-{stack}-developer
description: >
  Follow {project_name} conventions for all code generation — {layer-1}, {layer-2}, {layer-3}.
  Always active when working in this repository.
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
- **{key architectural principle}**: {e.g., "Thin controllers: Controllers handle HTTP concerns. Business logic belongs in service objects."}

## {Layer 1 — e.g., Service Objects}

{Brief description of where this layer lives and its role.}

- {Rule 1 — e.g., "Inherit from ApplicationService and implement #call."}
- {Rule 2 — e.g., "Invoke with ClassName.call(params) — never ClassName.new(params).call."}
- {Rule 3 — e.g., "Wrap multi-step mutations in ActiveRecord::Base.transaction."}

```{language}
# Brief inline annotated example (~10-15 lines)
{example code with inline comments}
```

See [references/{layer-1}.md](references/{layer-1}.md) for annotated examples.

## {Layer 2 — e.g., Controllers}

{Brief description.}

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
# 1. {what rule this line shows}
# 2. {what this block demonstrates}
{annotated code excerpt — 20-40 lines max}
# ↑ {annotation explaining a key pattern}
```

### Key rules for this layer:

- {Rule 1 — matches annotation 1 above}
- {Rule 2 — matches annotation 2 above}
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
4. Annotate with inline comments explaining *why*, not just *what*
5. Write — do NOT copy entire files

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

*Why this works: {1-2 sentence explanation}*
```

Insert after the most relevant existing section. Do not append to the end of the file — place examples near related content.
