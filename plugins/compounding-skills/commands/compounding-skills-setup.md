---
name: compounding-skills-setup
description: One-time setup wizard that builds a personalized .claude/ library from your codebase — run once as /compounding-skills-setup and never again.
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
---

# compounding-skills-setup — Personalized .claude/ Setup Wizard

**Run this once after installing compounding-skills.** It builds a tailored `.claude/` library for your project — workflow commands, project skills, and specialized agents — based on how you actually code.

> **One-time command.** After setup completes, use your generated `{command_prefix}:compound` command to keep skills in sync. Never run `/compounding-skills-setup` again on the same project.
>
> **Prerequisite:** Run `/init` first to generate your `CLAUDE.md` file. This setup wizard focuses on workflow commands and expert skills — project context belongs in `CLAUDE.md`.

**Process knowledge:** Load the `skill-garden` skill for generator logic, codebase analysis techniques, and all output templates.

## Phase 1 — Detect Project Type

Auto-detect whether this is a brownfield (existing code) or greenfield (new) project:

```bash
# Count commits
git log --oneline -5 2>/dev/null | wc -l

# Check for project files
ls Gemfile package.json pyproject.toml tsconfig.json go.mod 2>/dev/null
```

**Brownfield** (≥5 commits OR recognized project files exist): Proceed to Phase 2A.
**Greenfield** (0–4 commits, no recognized project files): Proceed to Phase 2B.

Also detect the project name from the current directory:
```bash
basename $(pwd)
```

Detect which AI coding tool is running this command:

```bash
# Detect tool
tool="claude"
if [ -n "$CURSOR_TRACE_ID" ] || [ -n "$CURSOR_SESSION_ID" ]; then
  tool="cursor"
elif command -v cursor &>/dev/null && [ ! -d ".claude" ]; then
  tool="cursor"
fi
```

Set `{tool}` (`claude` or `cursor`) and `{config_dir}` (`.claude` for Claude Code, `.cursor` for Cursor CLI).

Then ask the user to choose their command prefix — this becomes the namespace for all workflow commands:

```
question: "What would you like to prefix your workflow commands with? This becomes the namespace (e.g., myapp:plan, myapp:work, myapp:brainstorm)."
header: "Command prefix"
options:
  - label: "{detected directory name} (Recommended)"
    description: "e.g., {detected_name}:plan, {detected_name}:work, {detected_name}:brainstorm"
  - label: "Short project alias"
    description: "Use an abbreviated name (you'll type it for every command)"
  - label: "Generic (no prefix)"
    description: "Commands named simply plan, work, brainstorm — use if this is your only project"
```

If the user selects "Short project alias" or wants a custom name, prompt for the exact string. Record as `{command_prefix}`.

**If `{tool}` is `cursor`:** Note after the prefix question: "Cursor CLI detected — commands will use underscore separator (e.g., `{prefix}_plan` instead of `{prefix}:plan`)."

Announce: "Detected [brownfield/greenfield] project. Detected tool: {tool}. Using `{command_prefix}` as command prefix. [Analyzing codebase / Running preference interview]..."

## Phase 2A — Brownfield: Codebase Analysis

**Process knowledge:** See `references/brownfield.md` for the full step-by-step analysis process.

Launch these Explore subagents **in parallel** to analyze the codebase:

<parallel_tasks>

#### 1. Tech Stack Detector
```
Task Explore: Identify the tech stack and framework versions.
Look for: Gemfile, package.json, pyproject.toml, go.mod, tsconfig.json, .ruby-version, .node-version, Cargo.toml.
Return: Primary language, framework + version, key dependencies, runtime version.
```

#### 2. Convention Mapper
```
Task Explore: Map file and directory conventions.
Look for: Where do controllers/services/components/models live?
What naming patterns exist (snake_case, camelCase, kebab-case)?
Any notable structural patterns (feature folders, domain-driven, etc.)?
Check: app/, src/, lib/, pkg/ directories.
Return: Directory structure map, naming conventions, key organizational patterns.
```

#### 3. Real Pattern Extractor
```
Task Explore: Find 2-3 representative files that best show coding patterns.
Avoid test files and auto-generated code.
Look for: A typical service/controller, a model/schema, a utility function.
Return: File paths + full content of the 2-3 most representative files.
```

#### 4. Tooling Inspector
```
Task Explore: Identify existing tooling setup.
Look for: Test runner (RSpec, Jest, pytest, go test), linter config (.rubocop.yml, .eslintrc, pyproject.toml [tool.ruff]), CI setup (.github/workflows/, .circleci/), formatter (prettier, black, gofmt).
Return: Test command, lint command, CI system, formatter.
```

#### 5. Git Workflow Analyzer
```
Task Explore: Analyze git workflow patterns.
Run: git log --oneline -20 2>/dev/null
Look for: Branch naming convention (feat/, fix/, chore/ prefixes), commit message style, PR/merge frequency.
Return: Branch naming pattern, commit message format, workflow style.
```

</parallel_tasks>

**Wait for all subagents to complete**, then compile findings into a structured summary:

```
Stack:        {language} {framework} {version}
Structure:    {key directories and naming}
Test runner:  {command}
Linter:       {command}
Git workflow: {pattern}
```

Display this summary to the user and announce: "Found your stack. Now let's configure your workflow commands."

## Phase 2B — Greenfield: Preference Interview

**Process knowledge:** See `references/greenfield.md` for the full question bank and rationale.

Use **AskUserQuestion** to ask these questions. Ask them **one at a time**.

**Question 1 — Language & Framework:**
```
question: "What's your primary language and framework?"
header: "Stack"
options:
  - label: "Ruby on Rails"
    description: "Rails — full stack web framework"
  - label: "TypeScript / Node.js"
    description: "TypeScript with Node, Next.js, or similar"
  - label: "Python"
    description: "Python with Django, FastAPI, or similar"
  - label: "Go"
    description: "Go — standard library or framework"
  - label: "PHP"
    description: "PHP with Laravel, Symfony, or similar"
```

**Question 2 — File Structure:**
```
question: "How do you prefer to organize files?"
header: "Structure"
options:
  - label: "By type (MVC)"
    description: "controllers/, models/, services/ — group by role"
  - label: "By feature/domain"
    description: "users/, orders/, payments/ — group by business domain"
  - label: "Flat with conventions"
    description: "Single directory with naming conventions"
  - label: "Framework default"
    description: "Whatever the framework suggests"
```

**Question 3 — Testing:**
```
question: "What's your testing approach?"
header: "Testing"
options:
  - label: "TDD — tests first"
    description: "Write tests before implementation"
  - label: "Tests after, high coverage"
    description: "Implement first, then write thorough tests"
  - label: "Integration tests only"
    description: "Focus on end-to-end and integration, skip unit tests"
  - label: "Minimal testing"
    description: "Test only critical paths"
```

**Question 4 — Git Workflow:**
```
question: "How do you work with git?"
header: "Git"
options:
  - label: "Feature branches + PRs"
    description: "Branch per feature, review via PR before merging"
  - label: "Trunk-based development"
    description: "Small commits directly to main, fast iteration"
  - label: "Gitflow"
    description: "main + develop + feature/release/hotfix branches"
  - label: "Solo / no strict workflow"
    description: "Commit when ready, no formal process"
```

Record answers as `greenfield_prefs` for use in Phase 3+.

## Phase 2C — Framework Deep-Dive Interview

Ask **stack-specific architecture questions** to drive the content of the expert developer skill. Use **AskUserQuestion** for each question.

**Brownfield pre-fill:** Before asking, check Phase 2A findings for auto-detectable answers (e.g., `pundit` in Gemfile → pre-select Pundit, `sidekiq` → pre-select Sidekiq). Present as confirmation questions — the user confirms and catches anything not auto-detected.

### Rails Projects

**Question 1 — Architectural Layers:**
```
question: "Which architectural layers does this Rails project use?"
header: "Layers"
multiSelect: true
options:
  - label: "Service objects"
    description: "Business logic in app/services/"
  - label: "Form objects"
    description: "Form validation in app/forms/"
  - label: "Presenters"
    description: "View logic in app/presenters/"
  - label: "Query objects"
    description: "Complex queries in app/queries/"
  - label: "Decorators"
    description: "Draper-style decorators in app/decorators/"
  - label: "Concerns"
    description: "Shared modules in app/models/concerns/ and app/controllers/concerns/"
  - label: "Serializers"
    description: "Response serialization in app/serializers/"
  - label: "Channels"
    description: "ActionCable channels in app/channels/"
  - label: "Mailers"
    description: "Email logic in app/mailers/"
  - label: "Background jobs"
    description: "Job classes in app/jobs/"
```

**Question 2 — Authorization:**
```
question: "Which authorization library does this project use?"
header: "Authorization"
options:
  - label: "Pundit"
    description: "Policy objects in app/policies/"
  - label: "CanCanCan"
    description: "Ability-based access control"
  - label: "Custom"
    description: "Project-specific authorization logic"
  - label: "None"
    description: "No authorization framework"
```

**Question 3 — Serialization:**
```
question: "How does this project serialize API responses?"
header: "Serialization"
options:
  - label: "Jbuilder"
    description: "JSON views in app/views/ (.json.jbuilder)"
  - label: "ActiveModel::Serializers"
    description: "Serializer classes"
  - label: "Blueprinter"
    description: "Blueprint classes"
  - label: "fast_jsonapi / jsonapi-serializer"
    description: "JSONAPI-compliant serializers"
  - label: "Plain Ruby"
    description: "Custom to_json or hash methods"
```

**Question 4 — Background Jobs:**
```
question: "Which background job system does this project use?"
header: "Background jobs"
options:
  - label: "Sidekiq"
    description: "Redis-backed background processing"
  - label: "Resque"
    description: "Redis-backed Resque jobs"
  - label: "Solid Queue"
    description: "DB-backed job queue (Rails 8+)"
  - label: "None"
    description: "No background job system"
```

**Question 5 — API Style:**
```
question: "What API style does this project use?"
header: "API style"
options:
  - label: "REST only"
    description: "Standard RESTful JSON API"
  - label: "GraphQL"
    description: "GraphQL API (graphql-ruby)"
  - label: "Mixed (REST + GraphQL)"
    description: "Both REST and GraphQL endpoints"
  - label: "Full-stack (HTML views)"
    description: "Server-rendered HTML with Hotwire/Turbo"
```

### TypeScript/Node Projects

**Question 1 — Framework Variant:**
```
question: "Which Node.js framework are you using?"
header: "Framework"
options:
  - label: "Express"
    description: "Classic Express.js"
  - label: "Fastify"
    description: "High-performance Fastify"
  - label: "NestJS"
    description: "Angular-inspired NestJS"
  - label: "Next.js (API routes)"
    description: "Next.js with API route handlers"
```

**Question 2 — ORM / DB:**
```
question: "Which ORM or database layer does this project use?"
header: "ORM/DB"
options:
  - label: "Prisma"
    description: "Type-safe ORM with schema-first approach"
  - label: "TypeORM"
    description: "Decorator-based ORM"
  - label: "Drizzle"
    description: "Lightweight TypeScript ORM"
  - label: "Sequelize"
    description: "Promise-based ORM"
  - label: "Raw SQL"
    description: "Direct SQL queries (pg, mysql2, etc.)"
```

**Question 3 — Validation:**
```
question: "Which validation library does this project use?"
header: "Validation"
options:
  - label: "Zod"
    description: "Schema-first TypeScript validation"
  - label: "Yup"
    description: "Schema builder for object validation"
  - label: "class-validator"
    description: "Decorator-based validation (common with NestJS)"
  - label: "None"
    description: "Custom or no validation library"
```

**Question 4 — Testing:**
```
question: "Which test framework does this project use?"
header: "Testing"
options:
  - label: "Jest"
    description: "Standard Jest testing"
  - label: "Vitest"
    description: "Vite-native testing framework"
  - label: "Other"
    description: "Custom or other test framework"
```

### Python Projects

**Question 1 — Framework:**
```
question: "Which Python web framework does this project use?"
header: "Framework"
options:
  - label: "Django"
    description: "Full-stack Django framework"
  - label: "FastAPI"
    description: "Modern async FastAPI"
  - label: "Flask"
    description: "Lightweight Flask"
```

**Question 2 — ORM:**
```
question: "Which ORM or database layer does this project use?"
header: "ORM"
options:
  - label: "Django ORM"
    description: "Built-in Django database layer"
  - label: "SQLAlchemy"
    description: "Python SQL toolkit and ORM"
  - label: "Tortoise ORM"
    description: "Async ORM inspired by Django ORM"
  - label: "None / Raw SQL"
    description: "Direct SQL or no ORM"
```

**Question 3 — Testing:**
```
question: "Which test framework does this project use?"
header: "Testing"
options:
  - label: "pytest"
    description: "Standard pytest"
  - label: "unittest"
    description: "Python built-in unittest"
```

### Go Projects

**Question 1 — Router:**
```
question: "Which HTTP router does this project use?"
header: "Router"
options:
  - label: "chi"
    description: "Lightweight, idiomatic chi router"
  - label: "gin"
    description: "High-performance Gin web framework"
  - label: "echo"
    description: "High performance Echo framework"
  - label: "stdlib (net/http)"
    description: "Standard library HTTP handlers"
```

**Question 2 — DB Layer:**
```
question: "Which database layer does this project use?"
header: "DB layer"
options:
  - label: "sqlc"
    description: "Type-safe SQL with generated code"
  - label: "GORM"
    description: "Go ORM"
  - label: "sqlx"
    description: "Extensions on database/sql"
  - label: "Raw database/sql"
    description: "Standard library database/sql"
```

### PHP Projects

**Question 1 — Framework:**
```
question: "Which PHP framework does this project use?"
header: "Framework"
options:
  - label: "Laravel"
    description: "Full-stack Laravel framework"
  - label: "Laravel Inertia"
    description: "Laravel with Inertia.js for frontend"
  - label: "Symfony"
    description: "Modular Symfony framework"
  - label: "CodeIgniter"
    description: "Lightweight CodeIgniter"
```

**Question 2 - Testing:**
```
question: "Which testing framework does this project use?"
header: "Testing"
options:
  - label: "PHPUnit"
    description: "Standard PHPUnit testing"
  - label: "Pest"
    description: "Elegant testing framework built on PHPUnit"
  - label: "Codeception"
    description: "Full-stack testing framework for PHP"
  - label: "None"
    description: "No testing framework"
```

**Question 3 - Frontend Framework:**
```
question: "Which frontend framework does this project use (if any)?"
header: "Frontend"
options:
  - label: "Inertia.js with React/Vue/Svelte"
    description: "Server-driven frontend with Inertia.js"
  - label: "Livewire"
    description: "Reactive components with Livewire"
  - label: "Blade + Alpine.js"
    description: "Server-rendered Blade templates with Alpine.js sprinkles"
  - label: "Separate SPA (React/Vue/Angular)"
    description: "Decoupled frontend application"
  - label: "None"
    description: "No frontend framework, server-rendered HTML only"
```


Record all answers as `expert_skill_config` for use in Phase 5.2.

## Phase 3 — Workflow Configuration

All 5 workflow commands (**brainstorm**, **plan**, **work**, **review**, **compound**) are always generated. This phase configures how they behave.

**Question 1 — Git workflow:**
```
question: "How do you work with git?"
header: "Git"
options:
  - label: "Feature branches + PRs (Recommended)"
    description: "Branch per feature, review via PR before merging"
  - label: "Trunk-based development"
    description: "Small commits directly to main, fast iteration"
  - label: "Solo / no strict workflow"
    description: "Commit when ready, no formal process"
```

**Question 2 — Work command behavior:**
```
question: "What should happen when /work finishes a task?"
header: "Work options"
options:
  - label: "Run tests + lint, then wait (Recommended)"
    description: "Quality checks run automatically; you decide when to commit"
  - label: "Run tests + lint + auto-commit"
    description: "Commit each completed task automatically"
  - label: "Run tests only"
    description: "Tests run; linting is manual"
  - label: "No automatic checks"
    description: "Implement only; quality checks are manual"
```

**Question 3 — Plan file location:**
```
question: "Where should plan files be saved?"
header: "Plan location"
options:
  - label: "docs/plans/ (Recommended)"
    description: "Standard location; commit plans with the codebase"
  - label: "tmp/plans/"
    description: "Keep plans local, outside version control"
```

Record all answers as `workflow_config` for use in Phase 5.

## Phase 4 — Agent Selection

Based on the detected/selected tech stack, suggest relevant agents:

**Stack-specific suggestions:**

| Stack | Suggested agents |
|-------|-----------------|
| Rails | dhh-rails-reviewer, kieran-rails-reviewer, data-integrity-guardian, schema-drift-detector |
| TypeScript | kieran-typescript-reviewer, julik-frontend-races-reviewer |
| Python | kieran-python-reviewer |
| Frontend | design-iterator, figma-design-sync |
| All | security-sentinel, performance-oracle, architecture-strategist, code-simplicity-reviewer |

Use **AskUserQuestion**:

```
question: "Which review agents do you want installed? (based on your {stack} stack)"
header: "Agents"
multiSelect: true
options:
  - label: "{stack-agent-1}"
    description: "{stack-agent-1 description}"
  - label: "security-sentinel"
    description: "Vulnerability scanning, auth review, OWASP compliance"
  - label: "performance-oracle"
    description: "N+1 queries, memory leaks, algorithmic complexity"
  - label: "architecture-strategist"
    description: "Design patterns, SOLID principles, separation of concerns"
  - label: "code-simplicity-reviewer"
    description: "YAGNI violations, over-engineering, unnecessary abstraction"
```

Then ask about custom agents:

```
question: "Do you want to describe any custom agents?"
header: "Custom agents"
options:
  - label: "No custom agents (Recommended)"
    description: "Use the standard agents selected above"
  - label: "Add a custom agent"
    description: "Describe an agent tailored to your project's specific needs"
```

If "Add a custom agent": prompt for name, description, and what it should focus on. Allow multiple. Record as `custom_agents`.

**Always generate:** The `code-simplifier` agent (see Phase 5 — it's always included).

## Phase 5 — Generate Files

**Process knowledge:** See `references/command-templates.md`, `references/skill-templates.md`, and `references/agent-templates.md` for all templates.

Announce: "Generating your personalized {config_dir}/ library..."

### 5.1 — Write Workflow Commands

**Claude Code** — write all 5 commands to `.claude/commands/{command_prefix}/` (subfolder per prefix):
- `.claude/commands/{command_prefix}/brainstorm.md`
- `.claude/commands/{command_prefix}/plan.md`
- `.claude/commands/{command_prefix}/work.md`
- `.claude/commands/{command_prefix}/review.md`
- `.claude/commands/{command_prefix}/compound.md`

**Cursor CLI** — write all 5 commands to `.cursor/commands/` (flat directory, underscore-separated filenames):
- `.cursor/commands/{command_prefix}_brainstorm.md`
- `.cursor/commands/{command_prefix}_plan.md`
- `.cursor/commands/{command_prefix}_work.md`
- `.cursor/commands/{command_prefix}_review.md`
- `.cursor/commands/{command_prefix}_compound.md`

Use the full templates from `references/command-templates.md`, substituting:

- `{command_prefix}` — the prefix chosen in Phase 1
- `{project_name}` — detected project name
- `{stack}` — detected/selected framework
- `{test_command}` — detected test runner command (brownfield) or inferred from stack (greenfield)
- `{lint_command}` — detected linter command
- `{plan_dir}` — plan directory from Phase 3 (default: `docs/plans/`)
- `{branch_style}` — branch naming pattern from Phase 3
- `{workflow_config}` — options selected in Phase 3

**Critical for the compound command:** In Phase 1.4 ("Read Current Skill Files"), list the actual skill files that will be created in Phases 5.2 and 5.3:
- `{config_dir}/skills/expert-{stack}-developer/SKILL.md`
- One line per reference file: `{config_dir}/skills/expert-{stack}-developer/references/{layer}.md` (one per layer from `expert_skill_config`)
- `{config_dir}/skills/expert-bug-hunter/SKILL.md`
- `{config_dir}/skills/expert-bug-hunter/references/techniques.md`

**Critical for the work command:** References to `expert-{stack}-developer` conventions (not a generic name) and `code-simplifier` agent.

**Brownfield only:** Include 1-2 real file path examples in plan and work (e.g., `app/services/example.rb:42`).

### 5.2 — Write Expert-{Stack}-Developer Skill

Write `{config_dir}/skills/expert-{stack}-developer/SKILL.md` using the Expert-Developer Skill template from `references/skill-templates.md`.

**SKILL.md frontmatter:**
```yaml
---
name: expert-{stack}-developer
description: >
  Follow {project_name} conventions for all code generation — {comma-separated list of layers from expert_skill_config}.
  Always active when working in this repository.
disable-model-invocation: false
---
```

**SKILL.md sections to write:**

1. **Architecture Overview** — project name, framework + version, API style, auth library, any project-specific identifiers or naming conventions
2. **Code Style** — linter/formatter (from Phase 2A), key style rules for the stack
3. **Per-layer sections** — one `##` section per layer selected in Phase 2C `expert_skill_config`. Each section includes:
   - 2-3 rules describing how this layer works in this project
   - A brief inline annotated example (~10-15 lines)
   - A link: `See [references/{layer}.md](references/{layer}.md) for annotated examples.`
4. **Error Handling** — error types → HTTP status mapping for the stack

**Then write one reference file per layer** at `{config_dir}/skills/expert-{stack}-developer/references/{layer}.md`:

**Brownfield:**
1. Use `Glob` / `Grep` to find 1-2 representative files for this layer (check `app/`, `src/`, `lib/`, `pkg/`)
2. Read them and select the most instructive ~20-40 lines — do NOT copy entire files
3. Annotate the excerpt with inline comments explaining key patterns
4. Write the reference file with the annotated example

**Greenfield:** Write a template example based on stack defaults, noting it's a starting point to be evolved via `/{command_prefix}:compound`.

**Reference file format** (see Template: Expert-Developer Skill in `references/skill-templates.md`).

### 5.3 — Write Expert-Bug-Hunter Skill

**Always generated.** Write three files at `{config_dir}/skills/expert-bug-hunter/`:

**`{config_dir}/skills/expert-bug-hunter/SKILL.md`** — A systematic debugging guide adapted for this project's stack and tooling. Include:
- Frontmatter: `name: expert-bug-hunter`, `description: Systematically debug {stack} issues — reproduces bugs, traces requests, inspects state, and identifies root causes`, `argument-hint: [bug description or error message]`, `disable-model-invocation: true`
- Phase 1: Triage — search codebase with Grep/Glob, check recent `git log`, locate specs. Reference actual spec directories from Phase 2A (e.g., `spec/controllers/`, `spec/services/`)
- Phase 2: Reproduce — write a minimal failing spec using the project's test runner (`{test_command}`)
- Phase 3: Investigate — request tracing through the project's actual layer structure (routes → controller → service/model → response), state inspection using the stack's debugger tool, regression hunting with git bisect
- Phase 4: Diagnose — root cause + chain of events + proposed fix
- Link to `references/techniques.md` throughout for detailed how-tos
- Rules: always clean up debugger statements, use `{lint_command}` on any touched files, ask before destructive operations

**`{config_dir}/skills/expert-bug-hunter/README.md`** — Usage guide with:
- What it does (4-step description)
- Usage examples with realistic invocations for this project's domain
- Techniques table (technique → when to use)
- How it integrates with the other `{command_prefix}:` commands
- When to use it (good cases) vs. when to skip it

**`{config_dir}/skills/expert-bug-hunter/references/techniques.md`** — Detailed debugging how-tos, adapted for this project:

**Brownfield:** Use Phase 2A findings to populate real file paths:
- Reproduction spec templates using the project's actual test helper paths and auth patterns
- Debugger usage (binding.pry for Rails, debugger/console.log for others) with real file examples from the codebase
- RSpec bisect / test-framework-specific bisect workflow
- Git bisect workflow
- Request tracing checklist mapping the project's actual layers (routes file → controller dir → service/model dir → response layer)
- Common {stack} bug patterns observed in this type of codebase

**Greenfield:** Write stack-appropriate templates, noting they are starting points to be evolved via `/{command_prefix}:compound`.

### 5.5 — Write Code-Simplifier Agent

**Always generated.** Write `{config_dir}/agents/code-simplifier.md`.

**Process knowledge:** See `references/agent-templates.md` for the code-simplifier template.

The agent's `skills:` frontmatter must reference `expert-{stack}-developer` (not `{stack}-patterns`). The agent's conventions section must reference the project's actual expert skill name.

**Brownfield:** Build the agent from real codebase evidence:
1. Read the Pattern Extractor's 2-3 representative files
2. Identify complexity signals: function length, abstraction depth, variable names, comments
3. Infer opinions: prefers small functions vs. large? explicit vs. implicit? verbose vs. terse?
4. Find 1-2 spots in the code that represent "appropriate complexity" for this project
5. Write the agent with these real excerpts as "✅ This is the right complexity level" examples

**Greenfield:** Write the agent with stack-appropriate defaults and note that examples will be added by `/{command_prefix}:compound` as the codebase grows.

### 5.6 — Write Selected Review Agents

For each agent selected in Phase 4, write `{config_dir}/agents/{agent-name}.md` using the template from `references/agent-templates.md`.

For custom agents: write `{config_dir}/agents/{custom-name}.md` with the description and focus areas provided.

### 5.7 — Ask About Additional Skills

Use **AskUserQuestion** to ask if the user wants any custom skills beyond the baseline:

```
question: "Would you like to add any custom skills beyond the baseline (expert-{stack}-developer + expert-bug-hunter)?"
header: "Extra skills"
options:
  - label: "No, baseline is sufficient (Recommended)"
    description: "The two expert skills cover the essentials — more can be added via /{command_prefix}:compound"
  - label: "Yes, add a custom skill"
    description: "Describe a specialized skill — e.g., domain conventions, external API patterns, deployment procedures"
```

If "Yes": ask for the skill name and description. Generate `{config_dir}/skills/{skill-name}/SKILL.md` using the New Pattern Skill template from `references/skill-templates.md`. Allow multiple additions by asking again until the user is done.

### 5.8 — Create Directory Structure

Ensure all needed directories exist.

**Claude Code:**
```bash
mkdir -p .claude/commands/{command_prefix} \
          .claude/skills/expert-{stack}-developer/references \
          .claude/skills/expert-bug-hunter/references \
          .claude/agents
```

**Cursor CLI:**
```bash
mkdir -p .cursor/commands \
          .cursor/skills/expert-{stack}-developer/references \
          .cursor/skills/expert-bug-hunter/references \
          .cursor/agents
```

Create plan and brainstorm directories:
```bash
mkdir -p {plan_dir} docs/brainstorms docs/learnings
```

## Phase 6 — Summary

Display a summary of everything created.

**Claude Code summary:**
```
✓ Setup complete

Commands (.claude/commands/{command_prefix}/):
  ✓ brainstorm.md   → /{command_prefix}:brainstorm
  ✓ plan.md         → /{command_prefix}:plan
  ✓ work.md         → /{command_prefix}:work
  ✓ review.md       → /{command_prefix}:review
  ✓ compound.md     → /{command_prefix}:compound

Skills:
  ✓ .claude/skills/expert-{stack}-developer/SKILL.md
  ✓ .claude/skills/expert-{stack}-developer/references/{layer-1}.md
  ✓ .claude/skills/expert-{stack}-developer/references/{layer-2}.md
  ... (one reference file per architectural layer)
  ✓ .claude/skills/expert-bug-hunter/SKILL.md
  ✓ .claude/skills/expert-bug-hunter/README.md
  ✓ .claude/skills/expert-bug-hunter/references/techniques.md
  ... (any additional skills requested in Phase 5.7)

Agents:
  ✓ .claude/agents/code-simplifier.md  ← always included
  ✓ .claude/agents/{selected-agent}.md
  ... (for each selected agent)

Start with:
  /{command_prefix}:brainstorm   — explore a feature idea
  /{command_prefix}:plan         — create an implementation plan
  /{command_prefix}:compound     — update skills after shipping

Run /{command_prefix}:compound after each feature to keep skills in sync.
```

**Cursor CLI summary:**
```
✓ Setup complete

Commands (.cursor/commands/):
  ✓ {command_prefix}_brainstorm.md   → /{command_prefix}_brainstorm
  ✓ {command_prefix}_plan.md         → /{command_prefix}_plan
  ✓ {command_prefix}_work.md         → /{command_prefix}_work
  ✓ {command_prefix}_review.md       → /{command_prefix}_review
  ✓ {command_prefix}_compound.md     → /{command_prefix}_compound

Skills:
  ✓ .cursor/skills/expert-{stack}-developer/SKILL.md
  ✓ .cursor/skills/expert-{stack}-developer/references/{layer-1}.md
  ✓ .cursor/skills/expert-{stack}-developer/references/{layer-2}.md
  ... (one reference file per architectural layer)
  ✓ .cursor/skills/expert-bug-hunter/SKILL.md
  ✓ .cursor/skills/expert-bug-hunter/README.md
  ✓ .cursor/skills/expert-bug-hunter/references/techniques.md
  ... (any additional skills requested in Phase 5.7)

Agents:
  ✓ .cursor/agents/code-simplifier.md  ← always included
  ✓ .cursor/agents/{selected-agent}.md
  ... (for each selected agent)

Start with:
  /{command_prefix}_brainstorm   — explore a feature idea
  /{command_prefix}_plan         — create an implementation plan
  /{command_prefix}_compound     — update skills after shipping

Run /{command_prefix}_compound after each feature to keep skills in sync.
```
