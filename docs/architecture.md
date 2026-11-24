# Architecture

How the marketplace is put together. Why it's built this way is in `docs/adr/`.

Claude Code reads `.claude-plugin/marketplace.json` when someone runs `/plugin marketplace add adamchrabaszcz/claude-marketplace`, then installs a plugin from the folder its entry points to (`/plugin install <name>@adams-claude-marketplace`). A plugin is a folder of Markdown and JSON; nothing is built.

## Layout

| Path | What |
| --- | --- |
| `.claude-plugin/marketplace.json` | The marketplace: its name, owner and one entry per plugin (`name`, `source`, `description`) |
| `plugins/<name>/.claude-plugin/plugin.json` | A plugin's manifest: name, description, author, repository. No version: Claude Code uses the last commit that changed the plugin's folder |
| `plugins/symfony-developer/agents/` | An agent for Symfony 7 development |
| `plugins/github-steward/skills/deliver/` | A skill: taking a change from plan to a pull request ready to merge |
| `plugins/github-steward/skills/onboard/` | A skill: starting a repository set up this way, or auditing one |
| `plugins/github-steward/skills/go-public/` | A skill: taking a private repository public, after scanning everything that becomes visible |
| `plugins/github-steward/templates/` | The files `onboard` writes into a repository; `README.md` lists the placeholders and pinned versions |
| `.github/workflows/` | CI and the Claude review |
| `docs/` | This file and the decision records |

## The github-steward plugin and its actions

`github-steward`'s templates don't carry scripts: their `format`, `issues` and `release` workflows call shared actions from [`adamchrabaszcz/github-steward-actions`](https://github.com/adamchrabaszcz/github-steward-actions), pinned to a release's commit SHA. A change to how issues, pull requests or releases are checked happens there, then the pin in the templates moves to the new release.
