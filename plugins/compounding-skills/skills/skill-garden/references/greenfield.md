# Greenfield Preference Interview

Question bank and rationale for interviewing developers about their preferences when starting a new project.

## Goal

For greenfield projects (little or no existing code), extract enough preference signals to write `.claude/` files that feel personalized rather than generic.

## Interview Philosophy

- Ask **one question at a time** using AskUserQuestion
- Prefer **multiple choice** over open-ended — easier to answer, maps cleanly to templates
- **4 questions maximum** before generating — don't over-interview
- Frame as "quick setup" not "configuration" — keep it lightweight

## The 4 Core Questions

### Q1: Language & Framework

**Why ask:** Determines which stack-specific agents and patterns to use.

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

**Mapping:**
- Ruby on Rails → suggest dhh-rails-reviewer, kieran-rails-reviewer, data-integrity-guardian
- TypeScript → suggest kieran-typescript-reviewer, julik-frontend-races-reviewer
- Python → suggest kieran-python-reviewer
- Go → suggest architecture-strategist, performance-oracle

### Q2: File Organization Preference

**Why ask:** Shapes the directory structure conventions in the expert skill and /plan guidance.

```
question: "How do you prefer to organize files?"
header: "Structure"
options:
  - label: "By type (MVC)"
    description: "controllers/, models/, services/ — group by role"
  - label: "By feature/domain"
    description: "users/, orders/, payments/ — group by business domain"
  - label: "Framework default"
    description: "Whatever the framework suggests — I'll follow conventions"
  - label: "Flat with conventions"
    description: "Single directory with naming conventions (everything in src/ or lib/)"
```

**Mapping to skill content:**
- By type → document where types live in the expert skill's Architecture Overview
- By feature → note to keep feature code cohesive in the expert skill
- Framework default → reference framework conventions in the expert skill
- Flat → note naming conventions are the primary organization signal

### Q3: Testing Approach

**Why ask:** Informs the /work command's test-before-complete configuration, and shapes patterns skill.

```
question: "What's your testing approach?"
header: "Testing"
options:
  - label: "TDD — tests first"
    description: "Write failing test, then implement to make it pass"
  - label: "Tests after, high coverage"
    description: "Implement first, then write thorough tests before shipping"
  - label: "Integration tests focus"
    description: "Skip unit tests, focus on end-to-end and integration coverage"
  - label: "Minimal testing"
    description: "Test only critical or complex paths — pragmatic coverage"
```

**Mapping to /work config:**
- TDD → add "write test first" reminder to /work Phase 1
- Tests after, high coverage → configure "run tests before completing" in /work
- Integration → note in patterns skill which integration test patterns to use
- Minimal → note in CLAUDE.md that coverage isn't a hard requirement

### Q4: Git Workflow

**Why ask:** Informs /plan auto-commit, /work auto-commit, and branch naming in commands.

```
question: "How do you work with git?"
header: "Git"
options:
  - label: "Feature branches + PRs"
    description: "Branch per feature, review via PR before merging to main"
  - label: "Trunk-based development"
    description: "Small commits directly to main, fast iteration"
  - label: "Solo / no strict workflow"
    description: "Commit when ready, whatever branch makes sense at the time"
```

**Mapping:**
- Feature branches + PRs → /work creates branch automatically, /review targets PRs
- Trunk-based → /work commits directly, smaller task granularity suggested
- Solo → relaxed workflow, auto-commit optional but convenient

## Optional Follow-Up Questions

Only ask these if the core questions leave significant ambiguity:

### Q5: Complexity Preference (for code-simplifier agent)

```
question: "How do you prefer to write code?"
header: "Style"
options:
  - label: "Small, focused functions"
    description: "Each function does one thing. Extract aggressively."
  - label: "Pragmatic"
    description: "Extract when it genuinely helps readability, not dogmatically"
  - label: "Prefer readable over DRY"
    description: "A little duplication is fine if it makes the code clearer"
```

### Q6: Framework Convention Stance

```
question: "How do you feel about framework conventions?"
header: "Conventions"
options:
  - label: "Follow them strictly"
    description: "Lean into the framework's opinions, minimize custom patterns"
  - label: "Adapt as needed"
    description: "Start with conventions, deviate when there's a good reason"
  - label: "Build our own patterns"
    description: "Use the framework as a base but establish our own conventions"
```

## Mapping Preferences to Templates

After the interview, map answers to template parameters:

```
stack:           {Q1 answer}              → used in all templates
structure:       {Q2 answer}              → expert skill Architecture Overview
testing:         {Q3 answer}              → /work command, patterns SKILL.md
git_workflow:    {Q4 answer}              → /plan, /work commands
complexity:      {Q5 answer or "pragmatic"} → code-simplifier agent
conventions:     {Q6 answer or "adapt"}   → patterns SKILL.md
```

## Greenfield Defaults by Stack

When generating files for greenfield, use these defaults as starting points:

### Ruby on Rails
- Test command: `bundle exec rspec`
- Lint command: `bundle exec rubocop -A`
- Conventions: Standard Rails (MVC, fat models/thin controllers, service objects)
- Complexity: Pragmatic — Rails idioms over clever abstractions

### TypeScript / Node.js
- Test command: `npm test` (or `yarn test`)
- Lint command: `npm run lint`
- Conventions: TypeScript strict mode, async/await, explicit types over inference
- Complexity: Avoid wrapper types, prefer plain objects where possible

### Python
- Test command: `pytest`
- Lint command: `ruff check .`
- Conventions: PEP 8, type hints, dataclasses over dicts for structured data
- Complexity: Pythonic idioms, list comprehensions, context managers

### Go
- Test command: `go test ./...`
- Lint command: `golangci-lint run`
- Conventions: Standard library first, explicit error handling, short variable names in small scopes
- Complexity: Flat, explicit over clever

### PHP
- Test command: `vendor/bin/phpunit` (or `vendor/bin/pest`)
- Lint command: `vendor/bin/phpcs`
- Conventions: PSR standards, Laravel/Symfony conventions if using those frameworks
- Complexity: Pragmatic, prefer readability over DRY

## Note in Generated Skills

Always include this note in greenfield-generated skills:

```markdown
> **Note:** This skill was bootstrapped from stated preferences, not observed code.
> Run `{command_prefix}:compound` after each feature to replace these defaults with real examples
> from your actual codebase.
```
