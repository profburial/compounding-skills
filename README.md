<img src="logo.svg" alt="Compounding Skills" width="520"/>

A plugin that helps you create workflow skills, custom skills, and agents based on your preferences.

## Philosophy

Standard AI skills are authored (or copied) once and never learn how you code — your conventions, your complexity preferences, your patterns. Every project ends up with generic advice.

Compounding Skills takes a different approach: generate skills from your actual codebase, then keep them in sync as you ship. The more you build, the better your skills get.

## The Workflow

The plugin produces a full development loop:

```
brainstorm → plan → work → review → compound
```

| Skill | Usage | What it does |
|-------|-------|--------------|
| brainstorm | `/{prefix}:brainstorm` | Explore requirements and approaches before committing |
| plan | `/{prefix}:plan` | Turn a feature into a structured implementation plan |
| work | `/{prefix}:work` | Execute a plan with quality checks and auto-simplification |
| review | `/{prefix}:review` | Exhaustive code review using project-specific conventions |
| compound | `/{prefix}:compound` | Extract patterns from what was just built; update skills |

For existing projects, the setup wizard explores your code to find examples of common patterns and conventions to bake directly into your skills.

For new projects, the plugin will ask you various questions based on your language/framework of choice to lay a solid foundation to compound on.

## Installation

```
/plugin marketplace add profburial/compounding-skills
/plugin install compounding-skills
```

### Setup

Run the setup wizard once:

```
/init
/compounding-skills:setup
```

`/init` generates your `CLAUDE.md`. `/compounding-skills:setup` builds your personalized skills library on top of it.

## What Setup Produces

```
.claude/
├── skills/
│   ├── {prefix}-brainstorm/
│   │   └── SKILL.md                 ← explore requirements collaboratively
│   ├── {prefix}-plan/
│   │   └── SKILL.md                 ← structured implementation planning
│   ├── {prefix}-work/
│   │   └── SKILL.md                 ← execute plans with quality checks
│   ├── {prefix}-review/
│   │   └── SKILL.md                 ← exhaustive code review
│   ├── {prefix}-compound/
│   │   └── SKILL.md                 ← extract patterns, update skills
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

## Audit

After setup, you can audit your generated skills to verify they actually improve Claude's output. The audit uses the same evaluation playbook as Anthropic's official skill-creator plugin.

```
/compounding-skills:audit
```

The audit:

1. **Discovers** your generated skills, lets you pick which to evaluate
2. **Generates test cases** — realistic prompts that exercise what each skill provides
3. **Runs dual comparisons** — every test runs with the skill AND without it (baseline)
4. **Grades and benchmarks** — structured assertions, pass rates, timing, and token usage
5. **Opens an interactive viewer** — review outputs qualitatively alongside quantitative benchmarks
6. **Iterates** — improves based on your feedback, reruns until you're satisfied
7. **Optimizes descriptions** (optional) — tunes trigger accuracy so skills fire when they should

This answers a concrete question: *does this skill actually make Claude better at coding in your project, or is it just taking up context window space?*

## License

MIT
