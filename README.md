# agents-skills

Personal collection of [Cursor](https://cursor.com) agent **rules** and **skills** for spec-driven development across frontend and backend.

## Structure

```
.agents/
├── rules/                    # Persistent agent rules (.mdc)
│   ├── general/
│   │   └── spec-driven.mdc
│   ├── frontend/
│   │   ├── frontend-architect.mdc
│   │   ├── react.mdc
│   │   └── typescript.mdc
│   └── backend/              # (future)
└── skills/                   # Specialized agent skills (SKILL.md)
    ├── general/
    │   ├── product-manager/
    │   ├── solution-architect/
    │   └── spec-writer/
    ├── frontend/
    │   ├── frontend-engineer/
    │   └── ui-engineer/
    └── backend/              # (future)
```

Content is grouped by domain:

- **general** — Cross-cutting rules and skills (spec-driven workflow, product, architecture).
- **frontend** — React, TypeScript, UI, and frontend engineering.
- **backend** — Reserved for future backend rules and skills.

### Rules

Rules live in `.agents/rules/<domain>/` as `.mdc` files. They guide agent behavior globally or for specific file patterns (via `globs` in frontmatter).

### Skills

Skills live in `.agents/skills/<domain>/<name>/SKILL.md`. Each skill defines a focused role or workflow the agent can follow when relevant.

## Usage

Copy or symlink this repository into a project, or reference its contents when configuring Cursor rules and skills for your workspace.

## License

MIT — see [LICENSE](LICENSE).
