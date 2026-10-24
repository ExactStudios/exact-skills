# Exact Skills

A Claude Code and Cursor plugin marketplace with code-review skills for WordPress plugin development.

## Skills

| Skill | Claude Code | Cursor | What it does |
|-------|-------------|--------|--------------|
| Exact Review JS | `/exact-skills:exact-review-js <folder> <todo-file>` | `/exact-review-js <folder> <todo-file>` | Read-only audit of the JS/JSX/TS/TSX code in one folder. Records verified findings as Pending tasks in a TODO file. |
| Exact Review PHP | `/exact-skills:exact-review-php <folder> <todo-file>` | `/exact-review-php <folder> <todo-file>` | Read-only audit of the WordPress PHP code in one folder. Records verified findings as Pending tasks in a TODO file. |

Both skills follow the same steps from `plugins/exact-skills/shared/audit-procedure.md`; each skill keeps its own review criteria.

## Prerequisite: ponytail

The skills use [ponytail](https://github.com/DietrichGebert/ponytail) to choose the smallest fix for each finding, and stop with install instructions when it is missing.

Claude Code:

```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

Cursor:

```bash
git clone https://github.com/DietrichGebert/ponytail
node ponytail/scripts/cursor-hooks.js install
```

## Install in Claude Code

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

## Install in Cursor

Team: a Cursor admin opens **Dashboard → Plugins & MCPs → Add Marketplace → Import from Repo** and pastes `https://github.com/ExactStudios/exact-skills`. Each developer then opens **Customize**, finds **exact-skills** and selects **Install**, choosing project or user scope.

Single developer, without a team marketplace:

```bash
git clone https://github.com/ExactStudios/exact-skills
mkdir -p ~/.cursor/plugins/local
ln -s "$PWD/exact-skills/plugins/exact-skills" ~/.cursor/plugins/local/exact-skills
```

Then reload the Cursor window. Install the whole plugin, not single skill folders: the skills read `shared/audit-procedure.md` from the plugin.

## Develop

```bash
claude plugin validate .
claude plugin marketplace add ./
```
