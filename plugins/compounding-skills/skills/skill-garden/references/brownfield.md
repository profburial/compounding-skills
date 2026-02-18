# Brownfield Codebase Analysis

Step-by-step process for extracting real patterns from an existing codebase during `compounding-skills:setup` Phase 2A.

## Goal

Extract enough real information to write `.claude/` files that reference actual paths, use real conventions, and feel tailored — not generic.

## Step 1: Tech Stack Detection

Run these detection commands and record findings:

```bash
# Ruby/Rails
cat Gemfile 2>/dev/null | grep -E "^gem ['\"]rails['\"]" | head -3
cat .ruby-version 2>/dev/null

# Node/TypeScript
cat package.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('dependencies',{}).get('next') or d.get('dependencies',{}).get('express') or 'node')" 2>/dev/null
cat .node-version .nvmrc 2>/dev/null

# Python
cat pyproject.toml 2>/dev/null | grep -E "^(django|fastapi|flask)" || grep -r "^django\|^fastapi\|^flask" requirements.txt 2>/dev/null | head -3

# Go
head -5 go.mod 2>/dev/null

# Framework version
grep -r "RUBY_VERSION\|node-version\|python_requires" .github/ .tool-versions 2>/dev/null | head -5
```

**What to record:**
- Primary language
- Framework name and version (e.g., "Rails 7.2", "Next.js 14", "Django 4.2")
- 2-3 key dependencies that shape conventions (e.g., "Sidekiq", "Tailwind", "SQLAlchemy")

## Step 2: Directory Convention Mapping

```bash
# Top-level structure
ls -la

# For Rails
ls app/ 2>/dev/null

# For Node/TypeScript
ls src/ 2>/dev/null

# For Python
ls -d */ 2>/dev/null | head -20

# Find where the "meat" lives
find . -name "*.rb" -not -path "*/vendor/*" -not -path "*_spec.rb" | head -30 | sort
find . -name "*.ts" -not -path "*/node_modules/*" -not -name "*.test.ts" -not -name "*.spec.ts" | head -30 | sort
find . -name "*.py" -not -path "*/__pycache__/*" -not -name "test_*.py" | head -30 | sort
```

**Patterns to identify:**

| Pattern | Signal |
|---------|--------|
| MVC structure | `app/controllers/`, `app/models/`, `app/views/` |
| Service layer | `app/services/`, `src/services/`, `services/` |
| Feature folders | `src/users/`, `src/orders/`, `src/payments/` |
| Domain-driven | `lib/domain/`, `app/domains/` |
| Flat structure | Everything in `src/` or `lib/` with naming conventions |

**Naming convention signals:**
- File naming: `snake_case.rb`, `camelCase.ts`, `kebab-case.ts`
- Class naming: `UserService`, `user_service`, `UserPresenter`
- Test location: collocated (`user.test.ts`) or separate (`spec/user_spec.rb`)

## Step 3: Representative File Selection

Select 2-3 files that best show the project's coding style. Avoid:
- Test files
- Auto-generated files (schema.rb, db/migrate/, *.generated.ts)
- Config files
- Very short files (< 20 lines)
- Very large files (> 300 lines) — find a medium-complexity example

**Good candidates:**
- A typical service or use case class
- A model or entity with some business logic
- A controller or handler (not hello-world simple, not mega-controller complex)

**How to find them:**

```bash
# Rails: find medium-complexity services
find app/services -name "*.rb" -exec wc -l {} \; | sort -n | awk '$1 > 30 && $1 < 150' | head -10

# TypeScript: find meaningful service files
find src -name "*.ts" -not -name "*.test.ts" -not -name "index.ts" -exec wc -l {} \; | sort -n | awk '$1 > 30 && $1 < 150' | head -10

# Python: find meaningful module files
find . -name "*.py" -not -name "test_*.py" -not -name "__init__.py" -exec wc -l {} \; | sort -n | awk '$1 > 30 && $1 < 150' | head -10
```

Read the top 3 candidates and pick the most representative.

## Step 4: Tooling Detection

```bash
# Test runner
ls Rakefile Guardfile .rspec pytest.ini jest.config.js jest.config.ts vitest.config.ts go.sum 2>/dev/null
cat package.json 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('scripts',{}).get('test','not found'))" 2>/dev/null

# Linter config
ls .rubocop.yml .eslintrc .eslintrc.js .eslintrc.json .eslintrc.yaml pyproject.toml 2>/dev/null
cat pyproject.toml 2>/dev/null | grep -A2 "\[tool.ruff\]" | head -5

# CI system
ls .github/workflows/ .circleci/ .gitlab-ci.yml 2>/dev/null

# Formatter
ls .prettierrc .prettierrc.js .prettierrc.json 2>/dev/null
cat pyproject.toml 2>/dev/null | grep -A2 "\[tool.black\]" | head -3
```

**Derive the actual commands:**

| Tool | Detection | Typical command |
|------|-----------|-----------------|
| RSpec | `ls .rspec` | `bundle exec rspec` |
| Jest | `package.json scripts.test` | `npm test` or `yarn test` |
| pytest | `ls pytest.ini` or pyproject | `pytest` |
| Go test | `ls go.sum` | `go test ./...` |
| RuboCop | `ls .rubocop.yml` | `bundle exec rubocop` |
| ESLint | `ls .eslintrc*` | `npm run lint` |
| Ruff | `pyproject.toml [tool.ruff]` | `ruff check .` |

## Step 5: Git Workflow Analysis

```bash
# Recent commit messages — look for style
git log --oneline -20 2>/dev/null

# Branch naming — look for prefixes
git branch -a 2>/dev/null | head -20

# PR frequency — are there merge commits?
git log --oneline -30 2>/dev/null | grep -i "merge"
```

**Patterns to identify:**

| Signal | Pattern |
|--------|---------|
| Commits like `feat: ...`, `fix: ...`, `chore: ...` | Conventional commits |
| Commits like `Add user auth`, `Fix cart bug` | Imperative style |
| Many merge commits | Feature branch + PR workflow |
| Linear history | Squash merges or trunk-based |
| Branch names like `feat/user-auth`, `fix/email-bug` | Conventional branch naming |
| Branch names like `terry/add-login` | Username-based branches |

## Step 6: Complexity Calibration

Read the 2-3 representative files selected in Step 3. Look for:

**Function length distribution:**
- Most functions < 10 lines → prefers small, focused functions
- Mix of sizes → pragmatic, no strong preference
- Some 50+ line functions → okay with larger functions when needed

**Abstraction patterns:**
- Many wrapper classes/objects → high abstraction preference
- Direct calls, minimal indirection → low abstraction preference

**Error handling style:**
- Rails: `save!` vs `save` + error check; `rescue_from` vs inline rescue
- TypeScript: try/catch vs Result types vs `.catch()`
- Python: raise exceptions vs return (False, error_message)

**Comment density:**
- No comments → self-documenting code preference
- Comments on every method → documentation preference
- Comments only on complex logic → pragmatic

Record these as complexity signals for the `code-simplifier` agent.
