<img src="logo.svg" alt="Compounding Skills" width="520"/>

A plugin that helps you create workflow commands, custom skills, and agents based on your preferences.

## Philosophy

Standard AI skills are authored (or copied) once and never learn how you code — your conventions, your complexity preferences, your patterns. Every project ends up with generic advice.

Compounding Skills takes a different approach: generate skills from your actual codebase, then keep them in sync as you ship. The more you build, the better your skills get.

## The Workflow

The plugin produces a full development loop:

```
brainstorm → plan → work → review → compound
```

| Command | Claude Code | Cursor | What it does |
|---------|------------|--------|--------------|
| brainstorm | `/{prefix}:brainstorm` | `/{prefix}_brainstorm` | Explore requirements and approaches before committing |
| plan | `/{prefix}:plan` | `/{prefix}_plan` | Turn a feature into a structured implementation plan |
| work | `/{prefix}:work` | `/{prefix}_work` | Execute a plan with quality checks and auto-simplification |
| review | `/{prefix}:review` | `/{prefix}_review` | Exhaustive code review using project-specific conventions |
| compound | `/{prefix}:compound` | `/{prefix}_compound` | Extract patterns from what was just built; update skills |

For existing projects, the setup command explores your code to find examples of common patterns and conventions to bake directly into your skills.

For new projects, the plugin will ask you various questions based on your language/framework of choice to lay a solid foundation to compound on. 

## Installation

### Claude Code 

```
/plugin marketplace add profburial/compounding-skills
/plugin install compounding-skills
```

### Cursor

Coming soon!

### Setup

**Claude Code** — run the setup wizard once:

```
/init
/compounding-skills-setup
```

`/init` generates your `CLAUDE.md`. `/compounding-skills-setup` builds your personalized skills library on top of it.

**Cursor** — run the setup wizard once:

```
/compounding-skills-setup
```

The wizard detects Cursor automatically and writes everything to `.cursor/`.

## What Setup Produces

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
│   │   ├── SKILL.md                 ← always-on architecture rules
│   │   └── references/
│   │       └── {layer}.md           ← annotated real code per layer
│   └── expert-bug-hunter/
│       ├── SKILL.md                 ← systematic debugging workflow
│       ├── README.md
│       └── references/
│           └── techniques.md
└── agents/
    ├── code-simplifier.md           ← tailored to your complexity style
    └── {stack-specific reviewers}
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
│   │       └── {layer}.md
│   └── expert-bug-hunter/
│       ├── SKILL.md
│       ├── README.md
│       └── references/
│           └── techniques.md
└── agents/
    ├── code-simplifier.md
    └── {stack-specific reviewers}
```

## License

MIT
