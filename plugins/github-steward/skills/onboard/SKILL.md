---
name: onboard
description: Use when the user wants to start a new project or repository on GitHub ("let's start working on ABC", "create a repo for …", "set up a new repository"). Creates github.com/<owner>/<name>, private or public, already set up - AGENTS.md, CONTRIBUTING, docs and decision records, issue and PR templates, CI with pinned actions, the format check, parent issues that close themselves, releases from Conventional Commits, the Claude review, Dependabot and the GitHub settings - and proves it with a first pull request.
---

# Onboard: start a repository set up the way we work

The files come from the plugin's `templates/` folder (`${CLAUDE_PLUGIN_ROOT}/templates`, two levels up from this skill). Read its `README.md` first: it lists every placeholder and what changes for a public repository.

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
3. Read every file through once: it must read as written for this project. Cut what doesn't apply (no scopes yet, no stack to set up), don't leave it empty.
4. Check: `grep -rn '{{[A-Z_]*}}' .` finds nothing, and every `.yml` under `.github/` parses.
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
2. A branch, a small change (with its test if there's code), a PR whose title is a Conventional Commit and whose description ends with `Closes #<issue>`.
3. Watch the checks: `test` and `pull-request` pass; `review` runs and posts a checklist. A review job that fails at the start usually means the app or the token is missing (step 5).
4. Merging is the user's. A docs PR makes no release; the first `feat` or `fix` will.

## 7. Report

Tell the user, in plain words: the repository's link; what was set up; anything skipped and why (a ruleset on a private repository, Codecov); what waits on them (merging the first PR). From here, changes follow the `deliver` skill.

## Never

- Create a repository, change settings or visibility, or delete anything without showing the commands and getting a yes.
- Ask for a token in the conversation, or write one into a file.
- Leave a placeholder, a "TODO" or an empty section in the first commit.
