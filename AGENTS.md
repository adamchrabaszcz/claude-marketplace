# Claude marketplace: agent instructions

A public Claude Code plugin marketplace, `adams-claude-marketplace`: plugins under `plugins/<name>/`, listed in `.claude-plugin/marketplace.json`. Plugins are Markdown skills and agents with JSON manifests, plus files a skill writes (`plugins/github-steward/templates/`).

This file is the rulebook for coding agents. The details are in `CONTRIBUTING.md`, the decisions in `docs/adr/`, the structure in `docs/architecture.md`; link to those instead of copying them here.

## Guardrails

1. Never commit or push directly to `main`. Branch and open a pull request.
2. An agent never merges a pull request. Merging is a human action.
3. Never rewrite shared history (`push --force`, rebasing `main`). To update a branch from `main`, merge `main` into it.
4. Ask before any outward-facing action beyond pushing a branch and opening its pull request: comments, creating or editing issues, releases, tags, repository settings.
5. No secrets in the repository, not even as examples. CI's tokens are GitHub Actions secrets.
6. No personal data in skills, agents, templates, issues or commits; examples use made-up names and `@example.com` addresses.

## Decide on your own

- **Branch name:** `<type>/<short-topic>` from the latest `main`.
- **Commit and pull request title:** a Conventional Commit, scoped by plugin (`feat(github-steward): …`).
- **Docs the change makes wrong:** update them in the same pull request (the README's plugin section, `docs/`).
- **Working notes and plans:** in `.plan/` (ignored by git).

## Ask first

- A new plugin, or renaming one: people install plugins by name, so a rename breaks their install.
- A change to what a skill does on GitHub (creating, changing or deleting things).
- Changes to CI, releases or repository settings beyond what the issue asks.
- A decision that deserves an ADR (`docs/adr/`): propose it, don't settle it alone.

## Workflow

1. Every change starts from a GitHub issue (format in `CONTRIBUTING.md`). One issue, one branch, one pull request.
2. Make the change with its docs; run the check below before every push.
3. The pull request follows `.github/pull_request_template.md` and ends with `Closes #<issue>`. It's squash merged, so its title becomes the commit on `main`.

## Checks

```sh
claude plugin validate .
```

## Code rules that bite

- **Manifests:** every plugin is listed in `.claude-plugin/marketplace.json` and has `.claude-plugin/plugin.json`; `claude plugin validate .` passes.
- **Reserved names:** plugin names don't start with `claude-` or `anthropic-`.
- **No `version`** in a plugin's `plugin.json` or marketplace entry: plugins follow `main` (`docs/adr/0002-plugins-follow-main.md`). A `version` would freeze every user on it until someone bumps it.
- **Generic content:** skills, agents and templates are for anyone; nothing from a particular project, person or machine.
- **Skills:** frontmatter with `name` and a `description` saying when to use it; the body is steps Claude can follow in the middle of a task, with the exact commands.
- **Templates:** every `{{PLACEHOLDER}}` is listed in `templates/README.md`; every `uses:` is pinned to a commit SHA with the version in a comment.

## Code review

Claude reviews each pull request when it's opened or marked ready (`.github/workflows/claude-review.yml`); `@claude review` in a comment asks for another pass (only the owner, members and collaborators can start it). When reviewing, look first at what CI can't see:

- **Generic content:** a project's name, path or setting in a skill, agent or template.
- **Skill triggers:** a description that would fire on the wrong requests, or miss the ones it's for.
- **Safety in skills:** an instruction that changes GitHub (repositories, settings, deletions) without showing the command and asking first, or that handles a token in the conversation.
- **Templates:** a placeholder missing from `templates/README.md`, a `uses:` not pinned, or a template that wouldn't read as written for the project once filled.
- **No personal data or secrets** in skills, agents, templates or examples.

Label each finding **blocking**, **should fix** or **nit**. Skip what the checks already catch. The summary is a checklist of these points and the definition of done in `CONTRIBUTING.md`, then the verdict. If nothing important turns up, keep it short.
