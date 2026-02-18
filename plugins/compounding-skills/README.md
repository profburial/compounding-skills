# compounding-skills

> Skills that compound with your codebase.

The compound-engineering-plugin provides excellent generic workflow commands and a large library of agents. But those skills and agents are authored once and never evolve — they don't know *how you code*.

**compounding-skills** fixes this: it generates a personalized `.claude/` setup tailored to your project's actual patterns, then keeps those skills in sync as your codebase evolves.

## Commands

### `/compounding-skills-setup` — One-time Setup Wizard

Run **once** after installing the plugin. Builds your `.claude/` library from scratch. Never run again on the same project.

- **Brownfield projects**: Analyzes real code to extract patterns, conventions, and examples
- **Greenfield projects**: Interviews you about your preferences

Generates:
- Workflow commands (`brainstorm`, `plan`, `work`, `review`, `compound`) under your chosen prefix (e.g., `myapp:plan`)
- An `expert-{stack}-developer` skill with per-layer conventions and real code references
- An `expert-bug-hunter` skill with systematic debugging workflows for your stack
- A `code-simplifier` agent built from real code examples showing your complexity preferences
- Stack-specific review agents (Rails, TypeScript, Python, etc.)

After setup, use your generated `{prefix}:compound` command to keep skills in sync as your codebase evolves.

## Installation

```
/plugin marketplace add https://github.com/profburial/compounding-skills
/plugin install compounding-skills
```

Then run the one-time setup wizard:

```
/init
/compounding-skills-setup
```

Run `/init` first to generate your `CLAUDE.md`. Then run `/compounding-skills-setup` to build your personalized `.claude/` library on top of it. Never run `/compounding-skills-setup` again on the same project.

## How It Works

```
Install → /compounding-skills-setup (once, never again)
             ↓
         Analyze codebase OR interview you
             ↓
         Write tailored .claude/ files
             ↓
         Code, ship features
             ↓
         {prefix}:compound (after each feature)
             ↓
         Skills update to reflect new patterns
             ↓
         Skills compound with your codebase
```

## The Key Differentiator

Standard skills are generic. `compounding-skills` generates skills that contain:

- **Real code excerpts** from your project as examples
- **Your actual conventions** (not hypothetical ones)
- **Your complexity preferences** (inferred from code style)
- **Evolution over time** as patterns are reinforced or updated

The `code-simplifier` agent, for example, uses actual before/after refactors from your codebase — not generic advice.

## Files Generated

After running `/compounding-skills-setup`, your project will have:

```
.claude/
├── commands/
│   └── {prefix}/
│       ├── brainstorm.md
│       ├── plan.md
│       ├── work.md
│       ├── review.md
│       └── compound.md
├── skills/
│   ├── expert-{stack}-developer/
│   │   ├── SKILL.md                    ← Always-on architecture rules
│   │   └── references/
│   │       ├── {layer-1}.md            ← Annotated real code per layer
│   │       └── {layer-2}.md
│   └── expert-bug-hunter/
│       ├── SKILL.md                    ← Systematic debugging workflow
│       ├── README.md
│       └── references/
│           └── techniques.md
└── agents/
    ├── code-simplifier.md              ← Tailored to your complexity style
    └── {other agents}.md
```
