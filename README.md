# Exact Skills

A Claude Code plugin marketplace with code-review skills for WordPress plugin development.

## Skills

| Skill | Command | What it does |
|-------|---------|--------------|
| Exact Review JS | `/exact-skills:exact-review-js <folder> <todo-file>` | Read-only audit of the JS/JSX/TS/TSX code in one folder. Records verified findings as Pending tasks in a TODO file. |
| Exact Review PHP | `/exact-skills:exact-review-php <folder> <todo-file>` | Read-only audit of the WordPress PHP code in one folder. Records verified findings as Pending tasks in a TODO file. |

Both skills follow the same steps from `plugins/exact-skills/shared/audit-procedure.md`; each skill keeps its own review criteria.

## Install

Add the marketplace once:

```bash
claude plugin marketplace add ExactStudios/exact-skills
```

Install for yourself, in every project:

```bash
claude plugin install exact-skills@exact-skills
```

Install for one project only (written to the project's `.claude/settings.json`, so teammates get it too):

```bash
claude plugin install exact-skills@exact-skills --scope project
```

The review skills load the `ponytail` plugin's skill for choosing fixes when it is installed, and fall back to the same rules when it isn't.

## Develop

```bash
claude plugin validate .
claude plugin marketplace add ./
```
