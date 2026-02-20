# compounding-skills

> Skills that compound with your codebase.

The compound-engineering-plugin provides excellent generic workflow commands and a large library of agents. But those skills and agents are authored once and never evolve — they don't know *how you code*.

**compounding-skills** fixes this: it generates a personalized setup tailored to your project's actual patterns, then keeps those skills in sync as your codebase evolves. Works with both **Claude Code** (writes to `.claude/`) and **Cursor** (writes to `.cursor/`).

## Commands

### Setup Wizard — One-time Setup

Run **once** after installing the plugin. Builds your config library from scratch. Never run again on the same project.

Command syntax differs by tool:

| Tool | Setup command | Workflow commands |
|------|--------------|-------------------|
| Claude Code | `/compounding-skills-setup` | `/{prefix}:brainstorm`, `/{prefix}:plan`, etc. |
| Cursor | `/compounding-skills_setup` | `/{prefix}_brainstorm`, `/{prefix}_plan`, etc. |

- **Brownfield projects**: Analyzes real code to extract patterns, conventions, and examples
- **Greenfield projects**: Interviews you about your preferences

Generates:
- Workflow commands (`brainstorm`, `plan`, `work`, `review`, `compound`) under your chosen prefix
- An `expert-{stack}-developer` skill with per-layer conventions and real code references
- An `expert-bug-hunter` skill with systematic debugging workflows for your stack
- A `code-simplifier` agent built from real code examples showing your complexity preferences
- Stack-specific review agents (Rails, TypeScript, Python, etc.)

After setup, use your generated `{prefix}:compound` (or `{prefix}_compound`) command to keep skills in sync as your codebase evolves.

## Installation

**Claude Code:**

```
/plugin marketplace add https://github.com/profburial/compounding-skills
/plugin install compounding-skills
```

Then run the one-time setup wizard:

```
/init
/compounding-skills-setup
```

Run `/init` first to generate your `CLAUDE.md`. Then run `/compounding-skills-setup` to build your personalized `.claude/` library on top of it.

**Cursor:**

```
/plugin install compounding-skills
/compounding-skills_setup
```

Note: `/init` is Claude Code specific. In Cursor, the setup wizard handles project context directly — just run `/compounding-skills_setup` to start.

Never run the setup command again on the same project.

## How It Works

```
Install → setup command (once, never again)
             ↓
         Detect tool (Claude Code or Cursor)
             ↓
         Analyze codebase OR interview you
             ↓
         Write tailored config files (.claude/ or .cursor/)
             ↓
         Code, ship features
             ↓
         {prefix}:compound (after each feature)
             ↓
         Skills update to reflect new patterns
             ↓
         Skills compound with your codebase
```

The wizard auto-detects whether you're running in Claude Code or Cursor and writes all output to the correct directory (`.claude/` or `.cursor/`).

## The Key Differentiator

Standard skills are generic. `compounding-skills` generates skills that contain:

- **Real code excerpts** from your project as examples
- **Your actual conventions** (not hypothetical ones)
- **Your complexity preferences** (inferred from code style)
- **Evolution over time** as patterns are reinforced or updated

The `code-simplifier` agent, for example, uses actual before/after refactors from your codebase — not generic advice.

## Files Generated

After running the setup wizard, your project will have:

**Claude Code** (`.claude/`):

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

**Cursor** (`.cursor/`):

```
.cursor/
├── commands/
│   ├── {prefix}_brainstorm.md
│   ├── {prefix}_plan.md
│   ├── {prefix}_work.md
│   ├── {prefix}_review.md
│   └── {prefix}_compound.md
├── skills/
│   ├── expert-{stack}-developer/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── {layer-1}.md
│   │       └── {layer-2}.md
│   └── expert-bug-hunter/
│       ├── SKILL.md
│       ├── README.md
│       └── references/
│           └── techniques.md
└── agents/
    ├── code-simplifier.md
    └── {other agents}.md
```
