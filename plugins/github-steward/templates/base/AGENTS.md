# {{PROJECT}}: agent instructions

{{DESCRIPTION}} {{STACK}}.

This file is the rulebook for coding agents. The details are in `CONTRIBUTING.md`, the decisions in `docs/adr/`, the structure in `docs/architecture.md`; link to those instead of copying them here.

## Guardrails

1. Never commit or push directly to `main`. Branch and open a pull request.
2. An agent never merges a pull request. Merging is a human action.
3. Never rewrite shared history (`push --force`, rebasing `main`). To update a branch from `main`, merge `main` into it.
4. Ask before any outward-facing action beyond pushing a branch and opening its pull request: comments, creating or editing issues, releases, tags, repository settings.
5. No secrets in the repository, not even as examples. CI's tokens are GitHub Actions secrets.
6. No personal data in code, tests, examples, issues or commits; fixtures use made-up names and `@example.com` addresses.

## Decide on your own

- **Branch name:** `<type>/<short-topic>` from the latest `main`.
- **Commit and pull request title:** a Conventional Commit; the type decides the release (`CONTRIBUTING.md`).
- **Tests:** every change adds or updates a test that fails without it.
- **Docs the change makes wrong:** update them in the same pull request.
- **Working notes and plans:** in `.plan/` (ignored by git), never in the repository; what comes out of them goes into ADRs and issues (`docs/adr/0002`).

## Ask first

- A new dependency or tool.
- A change people already rely on (a format, an interface, a default): that's a breaking change.
- Changes to CI, releases or repository settings beyond what the issue asks.
- A decision that deserves an ADR (`docs/adr/`): propose it, don't settle it alone.
- Work that grows past its issue: suggest a follow-up issue instead.

## Workflow

1. Every change starts from a GitHub issue (format in `CONTRIBUTING.md`). One issue, one branch, one pull request.
2. Make the change with its tests and docs; run the checks below before every push.
3. The pull request follows `.github/pull_request_template.md` and ends with `Closes #<issue>`. It's squash merged, so its title becomes the commit on `main`.

## Checks

```sh
{{SETUP_COMMAND}}
{{TEST_COMMAND}}
```

## Code rules that bite

{{CODE_RULES}}
- **Modular code (`docs/adr/0003`):** one job per module and function, settings passed in with defaults, I/O at the edges, one source per shape; split a file before it keeps growing.
- GitHub Actions are pinned to a full commit SHA with the version in a comment.

## Code review

Claude reviews each pull request when it's opened or marked ready (`.github/workflows/claude-review.yml`); `@claude review` in a comment asks for another pass (only the owner, members and collaborators can start it). When reviewing, look first at what CI can't see:

{{REVIEW_POINTS}}
- **Modular code (`docs/adr/0003`):** a module or function doing several jobs, a hard-coded setting that will be tuned, a shape written out twice, or I/O mixed into the logic.
- **No plan files (`docs/adr/0002`):** a pull request that adds a plan to the repository instead of ADRs and issues.
- **Tests:** a change without a test that fails without it.
- **No personal data or secrets** in code, tests, docs or examples.

Label each finding **blocking**, **should fix** or **nit**. Skip what the checks already catch. The summary is a checklist of these points and the definition of done in `CONTRIBUTING.md`, then the verdict. If nothing important turns up, keep it short.
