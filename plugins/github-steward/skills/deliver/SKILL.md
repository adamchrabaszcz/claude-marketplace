---
name: deliver
description: Use for any change to a GitHub repository - a feature, fix, docs, CI or refactor, or a request that bundles several. Takes each change from an approved plan to a pull request that's ready to merge - an issue per task, one branch and PR per change from the latest main, checks and a self-review before committing, Conventional Commits, a test merge of all open PRs before reporting, and clean-up after merges. Also use when the user says a PR was merged.
---

# Deliver changes: plan, issue, one small PR each

The repository's own `AGENTS.md`, `CONTRIBUTING.md` and `docs/` win wherever they differ from this skill. Read them first. If they don't exist, offer the `onboard` skill once and carry on with these defaults.

## 1. Plan before building

- **Something not fully specified** (a design choice, wording, where a control goes, a trade-off): write a short plan in `.plan/<topic>.md`. Make sure `.plan/` is git-ignored. List what will change and the decisions to make, give a recommendation for each, and **wait for the user's answers**. Record the answers in the plan.
- **Small, unambiguous fixes** don't need a plan. Say what you'll do and do it.
- **Decisions belong on the issue too**: add them as a comment, so they outlive the conversation.

## 2. Split the request, one issue each

- List the separate changes in the request. **Each one is one issue and one pull request.** Three things asked in one message means three PRs.
- Find the issue, or create it: `gh issue create --title "<plain words>" --label enhancement|bug|documentation --body "<what and why>"`.
- Ideas that aren't planned yet are issues too. The roadmap is the issue list, never a section in the docs.

## 3. Build one change

1. Start from the latest main: `git checkout main && git pull`, then `git checkout -b <type>/<short-topic>`. Types: `feat`, `fix`, `docs`, `ci`, `refactor`, `test`, `perf`, `build`, `chore`.
2. **Keep pull requests independent.** Never branch from another open branch. If this change needs one that isn't merged yet, say so and wait for that merge. When two open PRs must touch the same file, put their changes in different places, e.g. next to the code each extends, not both appended at the end.
3. Make the change, and update its tests and docs in the same PR.
4. Run the repository's checks: the commands in AGENTS.md or CONTRIBUTING.md, otherwise what CI runs. For UI changes, also look at the result at desktop and phone width (headless browser screenshots if there's no other way).
5. **Review your own diff** before committing:
   - correctness and edge cases;
   - escaping and other security;
   - leftover debug code;
   - docs that still match the code;
   - nothing unrelated in the diff.
   Fix what you find.

## 4. Commit and open the pull request

- **Conventional Commits:** `type(optional scope)!: subject in the imperative`, a body saying why, and a footer `Closes #<issue>`, or `Part of #<issue>` for one step of several. The type matters: if the repository releases from commits, it decides the version (`feat` minor, `fix`/`perf` patch, `!` or a `BREAKING CHANGE:` footer major).
- Follow the repository's commit attribution rules, if it has any.
- Push the branch and open the PR with `gh pr create`. The body says:
  - **what changes**, in plain words a reviewer who wasn't there can follow;
  - **how it was checked**: tests, screenshots, real-environment runs;
  - **anything the user must do**: secrets to add, settings, merge order;
  - `Closes #<issue>`.

## 5. Before reporting back

- **Test-merge every open PR together:** a temporary branch from main, merge each open branch, run the checks, then delete the temporary branch. If two collide, move one change elsewhere in its own branch and push again. Don't stack branches.
- Tell the user in plain language:
  - what's ready, with links;
  - what each PR does;
  - what waits on them: merging, the order, secrets, settings, open questions.

## 6. After a merge

- `git checkout main && git pull`. Delete merged local branches, and remote ones too if GitHub doesn't delete them on merge (offer to switch that setting on).
- Check the issues the PR closed are closed, and pick up anything that was waiting for this merge.

## Never

- Commit or push to `main`, or force-push a shared branch.
- Bundle unrelated changes into one PR, or stack an unmerged branch on another.
- Write secrets into files, or ask the user to paste a secret into the conversation. Tokens go into repository secrets, which the user sets.
- Change outward-facing things without a clear yes, such as:
  - repository settings and branch protection;
  - visibility;
  - deleting branches or releases;
  - publishing.
