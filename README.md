# Compounding Skills 

A Claude Code plugin that helps you create workflow commands, custom skills, and agents based on your preferences. 

## Philosophy

Standard AI skills are authored (or copied) once and never learn how you code — your conventions, your complexity preferences, your patterns. Every project ends up with generic advice.

Compounding Skills takes a different approach: generate skills from your actual codebase, then keep them in sync as you ship. The more you build, the better your skills get.

## The Workflow

The plugin produces a full development loop:

```
brainstorm → plan → work → review → compound
```

- **`/{prefix}:brainstorm`** — explore requirements and approaches before committing
- **`/{prefix}:plan`** — turn a feature into a structured implementation plan
- **`/{prefix}:work`** — execute a plan with quality checks and auto-simplification built in
- **`/{prefix}:review`** — exhaustive code review using project-specific conventions
- **`/{prefix}:compound`** — extract patterns from what was just built; update skills

For existing projects, the setup command explores your code to find examples of common patterns and conventions to bake directly into your skills.

For new projects, the plugin will ask you various questions based on your language/framework of choice to lay a solid foundation to compound on. 

## Installation

```
/plugin marketplace add profburial/compounding-skills
/plugin install compounding-skills
```

Then run the setup wizard **once**:

```
/init
/compounding-skills-setup
```

`/init` generates your `CLAUDE.md`. `/compounding-skills-setup` builds your personalized `.claude/` library on top of it.

## What Setup Produces

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

## License

MIT
