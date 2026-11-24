# Contributing

Contributions are welcome: new plugins, fixes, ideas. By contributing you agree that your work is released under the repository's [MIT licence](LICENSE). Security problems go through `SECURITY.md`, not issues.

Where things are written down: `AGENTS.md` (the short version, also for coding agents), `docs/architecture.md` (structure), `docs/adr/` (decisions, numbered, never deleted).

## Workflow

1. Find or open an issue for the change (format below).
2. Branch from the latest `main`: `<type>/<short-topic>`, e.g. `fix/export-file-name`.
3. Make the change, focused on one thing, with the docs it makes wrong.
4. Run the checks (below), then open a pull request into `main`. CI must be green before merging. Claude reviews it once it's opened or marked ready; `@claude review` in a comment asks for another pass.
5. Pull requests are squash merged: the title becomes the commit on `main` and the description its body. So the title is a Conventional Commit and the description ends with `Closes #<issue>`.

### One issue per pull request

- Each change gets its own branch and pull request, even when several are asked for at once. Each starts from `main` and can be merged on its own.
- Work that takes several pull requests is a parent issue with a GitHub sub-issue per step; each pull request closes its sub-issue, and the parent closes itself when the last one does.
- If `main` moves and a pull request conflicts, merge `main` into the branch. No force-pushes.

### Definition of done

- [ ] it closes exactly one issue, and every "Done when" item there is met
- [ ] `claude plugin validate .` passes, and a changed skill was tried on a real task
- [ ] the checks pass locally and CI is green
- [ ] docs it makes wrong are updated
- [ ] a decision that is costly to reverse has an ADR in `docs/adr/`
- [ ] no personal data and no secrets
- [ ] the title is a Conventional Commit of the right type, and the description ends with `Closes #<issue>`

## Commits: Conventional Commits

`<type>(<optional scope>)<optional !>: <summary>`, summary in the imperative, lowercase, no full stop.

| Type | For |
| --- | --- |
| `feat` | something new people can use |
| `fix` | a bug fix |
| `perf` | faster, same behaviour |
| `docs`, `test`, `refactor`, `style`, `build`, `ci`, `chore` | everything else |

Scopes: the plugin's name (`github-steward`, `symfony-developer`).

A change people must adapt to gets a `!` after the type and a `BREAKING CHANGE:` footer saying what they need to do.

## Issues and pull requests

The roadmap is the issue list. Issues use one format; the issue forms write it, and when you write the body yourself, use the same headings:

```markdown
### What and why
What should change and why, in plain words. For a bug: what happens, what should, and how to see it.

### Done when
- [ ] A short checklist that says when it is finished

### Blast radius
Optional. What else it touches or could break, and who would notice.

### Parent
Optional. #n, when it is one step of bigger work (also add it as a sub-issue there).
```

Titles are plain words, no type prefix. `.github/workflows/format.yml` checks both: an issue missing "What and why" or "Done when" gets a `needs-format` label and a comment until it's fixed; a pull request without a Conventional Commit title or a `Closes #<issue>` line fails the check.

## Versions

Plugins have no `version` field: each follows `main` (`docs/adr/0002-plugins-follow-main.md`). Claude Code takes a plugin's version from the last commit that changed its folder, so merging a pull request that changes a plugin is its release, and `/plugin update` delivers it. Other plugins don't move. There are no tags or releases, and nothing to bump.

## Checks

```sh
claude plugin validate .                     # the marketplace and every plugin's manifest
claude --plugin-dir plugins/<name>           # try a plugin's skills and agents in a session
```

A change to a skill is also tried once by hand, on a real task, before the pull request: skills are instructions, and only a run shows whether Claude can follow them.

## Adding a plugin

1. A folder `plugins/<name>/` with `.claude-plugin/plugin.json` (name, description, author, repository; no `version`, see Versions). The name doesn't start with `claude-` or `anthropic-`, which are reserved.
2. Its skills (`skills/<skill>/SKILL.md`), agents, hooks or commands.
3. An entry in `.claude-plugin/marketplace.json` and a section in the README with what it does and how to install it.

## Conventions

- GitHub Actions are pinned to a full commit SHA with the version in a comment; Dependabot proposes updates. Its pull requests are the one exception to "every pull request closes an issue"; rename their titles to this style when merging (`ci: bump checkout to v7.0.1`).
- Comments explain *why*, not *what*.
