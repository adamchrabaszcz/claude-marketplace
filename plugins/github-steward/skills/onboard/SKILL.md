---
name: onboard
description: Use when the user wants to start a new project or repository on GitHub ("let's start working on ABC", "create a repo for …", "set up a new repository"), or to check an existing repository's setup ("audit this repo", "what's missing here", "bring this repo up to our setup"). Creates github.com/<owner>/<name>, private or public, already set up - AGENTS.md, CONTRIBUTING, docs and decision records, issue and PR templates, CI with pinned actions, the format check, parent issues that close themselves, releases from Conventional Commits, the Claude review, Dependabot and the GitHub settings - and proves it with a first pull request. For an existing repository, lists what's missing or out of date and fills each gap after a yes.
---

# Onboard: start a repository set up the way we work, or audit one

The files come from the plugin's `templates/` folder (`${CLAUDE_PLUGIN_ROOT}/templates`, two levels up from this skill). Read its `README.md` first: it lists every placeholder and what changes for a public repository.

**A new repository:** steps 1 to 7. **An existing one:** "Audit an existing repository" at the end; it never overwrites, it reports and then fills one gap at a time.

Every command that creates or changes something on GitHub is shown to the user before it runs, grouped so one yes covers a step.

## 1. Check, then ask only what you can't tell

1. `gh auth status` must show a login with the `repo` scope. The owner is `gh api user --jq .login` unless the user names an organisation.
2. Ask, in one message (use the question tool where it has one), only what the request doesn't already say:
   - **Name:** the repository name, lowercase with hyphens (`abc`); the display name as written in prose (`ABC`).
   - **One sentence:** what it is and who it's for.
   - **Private or public.** Private uses the account's free Actions minutes (2,000 a month on the free plan; CI and the Claude review both count) and can't have a branch ruleset on the free plan. Public has neither limit, but everything in it is visible.
   - **Stack**, or "not yet".
   - **Public only:** the licence (MIT unless they say otherwise) and the copyright holder; anything kept out of the licence, such as the name and logo.
3. `gh repo view <owner>/<name>` must fail (the name is free). Ask where to clone it if it isn't obvious (usually next to the user's other projects).
4. Draft `{{CODE_RULES}}` and `{{REVIEW_POINTS}}` from the stack and the description: three to five bullets each, the things that would hurt if broken. Show them with the plan in the next step; they grow with the project.

## 2. Create the repository

Show, then run:

```sh
gh repo create <owner>/<name> --private|--public --description "<one sentence>" --clone
```

Run it from the folder it should be cloned into. Don't let it add a licence or README: they come from the templates.

## 3. The first commit

This is the one commit that goes straight to `main`: a new repository has nothing to branch from. Everything after it goes through issues and pull requests.

1. Copy `templates/base/` into the repository, then `templates/public/` on top for a public one (its files replace base ones at the same path).
2. Fill every `{{PLACEHOLDER}}`. GitHub's own `${{ … }}` expressions stay. For a public repository, add the licence section to the README and the welcome paragraph to CONTRIBUTING.md (templates README).
3. Read every file through once: it must read as written for this project. Cut what doesn't apply, don't leave it empty. The usual case is **a stack but no code yet**:
   - no setup command: drop that line from the command blocks in AGENTS.md, CONTRIBUTING.md and the README;
   - no package manifest yet: drop the package ecosystem from `dependabot.yml` (it fails on a missing manifest) and add it with the first dependency;
   - the test step runs the stack's test command, which passes with no tests (`node --test` does), or the "nothing to test yet" step from the templates README;
   - no scopes: drop the "Scopes:" line.
4. Check: `grep -rn '{{[A-Z_]*}}' .` finds nothing; no line is left as a bare `Scopes: .` and no code block is empty or starts with a blank line; every `.yml` under `.github/` parses.
5. Commit as `chore: repository setup: docs, CI and the Claude review` (with the user's attribution rules, if any) and push.

## 4. Settings

Show the whole batch, then run it. `R=repos/<owner>/<name>`.

```sh
# Squash merges only, the PR title and description becoming the commit; tidy branches
gh api -X PATCH $R -F allow_merge_commit=false -F allow_rebase_merge=false -F allow_squash_merge=true \
  -f squash_merge_commit_title=PR_TITLE -f squash_merge_commit_message=PR_BODY \
  -F delete_branch_on_merge=true -F allow_update_branch=true -F has_wiki=false
gh label create chore -R <owner>/<name> --color c5def5 --description "CI, tooling, dependencies; nothing changes for users"
gh api -X PUT $R/vulnerability-alerts && gh api -X PUT $R/automated-security-fixes
gh api -X PUT $R/topics -f "names[]=<topic>" …   # two to four, from the description
```

**Public only:**

```sh
gh api -X PUT $R/private-vulnerability-reporting
gh api -X PUT $R/actions/permissions/fork-pr-contributor-approval -f approval_policy=all_external_contributors
gh api -X PATCH $R --input - <<'EOF'
{"security_and_analysis": {"secret_scanning": {"status": "enabled"}, "secret_scanning_push_protection": {"status": "enabled"}}}
EOF
```

**The `main` ruleset**, public only (GitHub refuses rulesets on private repositories of the free plan). Required checks are the job names `test` and `pull-request`. "Branches up to date before merging" (`strict`) is asked, not assumed: with several open pull requests, each merge makes the others update again.

```sh
gh api -X POST $R/rulesets --input - <<'EOF'
{"name": "main", "target": "branch", "enforcement": "active",
 "conditions": {"ref_name": {"include": ["~DEFAULT_BRANCH"], "exclude": []}},
 "bypass_actors": [{"actor_id": 5, "actor_type": "RepositoryRole", "bypass_mode": "always"}],
 "rules": [{"type": "deletion"}, {"type": "non_fast_forward"},
  {"type": "pull_request", "parameters": {"required_approving_review_count": 0, "dismiss_stale_reviews_on_push": false,
    "require_code_owner_review": false, "require_last_push_approval": false, "required_review_thread_resolution": false,
    "allowed_merge_methods": ["squash"]}},
  {"type": "required_status_checks", "parameters": {"strict_required_status_checks_policy": false, "do_not_enforce_on_create": false,
    "required_status_checks": [{"context": "test"}, {"context": "pull-request"}]}}]}
EOF
```

## 5. What only the owner can do

Stop and ask for these two, with the link and the command. Without them, the Claude review job fails.

- **The Claude GitHub App must cover the repository.** If it's installed for "only select repositories", a new one isn't included: https://github.com/settings/installations → Claude → Configure → add it.
- **The `CLAUDE_CODE_OAUTH_TOKEN` secret.** Personal accounts have no account-wide secrets, so each repository gets its own copy of the same token. The user runs it, typing the value at the prompt; it never goes into the conversation:

  ```
  ! gh secret set CLAUDE_CODE_OAUTH_TOKEN -R <owner>/<name>
  ```

  One token serves every repository. A new one comes from `claude setup-token` (run in their own terminal) and lasts a year; suggest they note the date.

Then check: `gh secret list -R <owner>/<name>` lists it.

## 6. Prove it with a first pull request

A setup isn't done until a pull request has gone through it.

1. An issue in the repository's format: "README: how to run it", or the first real task if the user has one.
2. A branch, a small change (with its test if there's code), a PR whose title is a Conventional Commit and whose description ends with `Closes #<issue>`. New files go into the layout table in `docs/architecture.md` in the same PR; the review checks it.
3. Watch the checks: `test` and `pull-request` pass; `review` runs and posts a checklist. A review job that fails at the start usually means the app or the token is missing (step 5). Its findings are handled as in the `deliver` skill: real bugs fixed, polish brought to the user.
4. Merging is the user's. A docs PR makes no release; the first `feat` or `fix` will (`v0.1.0` unless `first-version` is set in `release.yml`), and the issue and PR get its milestone and a "Released in" comment.

## 7. Report

Tell the user, in plain words: the repository's link; what was set up; anything skipped and why (a ruleset on a private repository, Codecov); what waits on them (merging the first PR). From here, changes follow the `deliver` skill.

## Audit an existing repository

The same setup, checked against what the repository already has. Existing files and choices win unless the user says otherwise: a repository can do something differently on purpose (its ADRs or CONTRIBUTING.md may say why). Read AGENTS.md, CONTRIBUTING.md and `docs/` first.

### 1. Look, without changing anything

`R=repos/<owner>/<name>`; note whether it's public (`gh repo view --json visibility`). Read files from the default branch as it is on GitHub (`git fetch`, then `git show origin/main:<path>` or `git ls-tree -r --name-only origin/main`), not from the working tree: a local checkout can be on an old branch and report gaps that aren't there.

| Area | Look at | It's there when |
| --- | --- | --- |
| Agent and contributor docs | `AGENTS.md`, `CONTRIBUTING.md`, `README.md` | AGENTS.md has guardrails, checks and a "Code review" section; CONTRIBUTING has the workflow, definition of done, Conventional Commits and the issue format. No `CLAUDE.md` beside AGENTS.md (Claude Code reads AGENTS.md) |
| Decisions and structure | `docs/architecture.md`, `docs/adr/` | both exist; ADRs follow the README's template |
| Issue and PR templates | `.github/ISSUE_TEMPLATE/`, `pull_request_template.md` | forms write "What and why" and "Done when"; blank issues off; the PR template ends with `Closes #` |
| CI | `.github/workflows/test.yml` | runs the project's checks on pull requests and on `main` |
| Format, parent issues, releases | `format.yml`, `issues.yml`, `release.yml` | they call `adamchrabaszcz/github-steward-actions` (pinned); copied scripts doing the same job are a gap only if the user wants to switch (they're older and lack the shared actions' fixes) |
| Claude review | `claude-review.yml`, `claude.yml`; `gh secret list -R …`; the latest review run | both workflows exist; the secret is set; a recent run on a pull request succeeded (a failure at the start means the app doesn't cover the repository or the token is missing) |
| Pinning and Dependabot | every `uses:` in `.github/`; `.github/dependabot.yml` | each `uses:` is a 40-character SHA with `# vX.Y.Z`; Dependabot watches the actions, and the packages for security fixes |
| Merge settings | `gh api $R --jq '{allow_squash_merge,allow_merge_commit,allow_rebase_merge,squash_merge_commit_title,squash_merge_commit_message,delete_branch_on_merge,allow_update_branch}'` | squash only, PR_TITLE and PR_BODY, branches deleted on merge |
| Security features | `gh api $R --jq .security_and_analysis`; `gh api $R/private-vulnerability-reporting`; `gh api $R/vulnerability-alerts` (204 = on) | Dependabot alerts on; **public:** secret scanning, push protection, private reporting on |
| Branch protection | `gh api $R/rulesets` | **public:** a ruleset on `main` requiring pull requests and the `test` and `pull-request` checks |
| Public files | `LICENSE`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `.github/CODEOWNERS` | **public:** all four, and the README has a licence section |

### 2. Report

One table: each area **in place**, **different on purpose** (with where that's written), or **missing**, with what would fill it. Then the gaps in a suggested order, settings first (they're quick and reversible), then one pull request per file area. Ask which to do; "none" is a fine answer.

### 3. Fill what the user picks

- **Settings:** show the commands from step 4 that the gap needs, then run them.
- **Files:** each gap is an issue and a pull request, as in the `deliver` skill. Start from the template, but fit it to the repository: keep what its existing files say, add what's missing, and fill `{{CODE_RULES}}` and `{{REVIEW_POINTS}}` from its real code. Never replace a file wholesale.
- **The Claude review:** the app and the secret are the owner's (step 5).

## Never

- Create a repository, change settings or visibility, or delete anything without showing the commands and getting a yes.
- Ask for a token in the conversation, or write one into a file.
- Leave a placeholder, a "TODO" or an empty section in the first commit.
- In an audit, overwrite an existing file or undo a choice the repository made on purpose.
