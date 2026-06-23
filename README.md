# agents-skills

Personal collection of [Cursor](https://cursor.com) agent **rules** and **skills** for spec-driven, frontend-focused development workflows.

## Structure

```
.agents/
├── rules/          # Persistent agent rules (.mdc)
│   ├── spec-driven.mdc
│   └── frontend/
└── skills/         # Specialized agent skills (SKILL.md)
    ├── product-manager/
    ├── solution-architect/
    ├── spec-writer/
    ├── ui-engineer/
    └── frontend-engineer/
```

### Rules

Rules live in `.agents/rules/` as `.mdc` files. They guide agent behavior globally or for specific file patterns (via `globs` in frontmatter).

### Skills

Skills live in `.agents/skills/<name>/SKILL.md`. Each skill defines a focused role or workflow the agent can follow when relevant.

## Usage

Copy or symlink this repository into a project, or reference its contents when configuring Cursor rules and skills for your workspace.

## License

MIT — see [LICENSE](LICENSE).
