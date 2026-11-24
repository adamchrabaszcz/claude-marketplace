---
name: go-public
description: Use when the user wants to make a private GitHub repository public ("open-source this repo", "make it public", "what's left before we go public"). Scans everything that becomes visible (the whole git history, every issue, pull request and comment, including edit history) for secrets, personal data and names that shouldn't be there; fills what a public repository needs (licence, security policy, code of conduct, code owners); checks deploy previews for fork pull requests; then, after the owner switches the visibility, turns on the public-only settings.
---

# Go public: take a private repository public, safely

Everything in the repository becomes public at once and for good: every commit on every branch, every issue, pull request and comment, their edit history, and workflow logs. A secret found afterwards has to be rotated; a name or email can't be taken back. So the scan comes first, and the switch is the owner's.

Every command that changes something is shown before it runs. `R=repos/<owner>/<name>`.

## 1. Ask what must not be there

Beyond secrets, ask the user once: names that mustn't appear (other projects, clients, employers, people), and whether real people's data was ever used (customers, competitors, riders, users). These become extra search terms.

## 2. Scan the git history

Fetch everything first (`git fetch --all --tags`), then search every commit on every branch, not just the working tree.

- **Secrets:** if `gitleaks` is installed, `gitleaks git --log-opts="--all"`. Otherwise:

  ```sh
  git log --all -p | grep -nEi '(api[_-]?key|secret|token|password|passwd|bearer|private[_-]?key)\s*[:=]|BEGIN [A-Z ]*PRIVATE KEY|gh[pousr]_[A-Za-z0-9]{20,}|github_pat_|sk-ant-[A-Za-z0-9-]{10,}|sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|xox[abpr]-'
  ```

  Read each hit: `${{ secrets.X }}`, test keys signed in the tests, and badge tokens meant to be public are fine; anything else is a finding.
- **Emails and authors:** `git log --all --format='%an <%ae>%n%cn <%ce>' | sort | uniq -c`, and `git log --all -p | grep -oEi '[a-z0-9._%+-]+@[a-z0-9.-]+\.[a-z]{2,}' | sort | uniq -c`. The owner's own address as commit author is normal (changing it means rewriting history); other real addresses in files are findings. `@example.com` is fine.
- **The extra terms** from step 1: `git log --all -p | grep -niE '<term>|<term>'`.
- **Files that were deleted** are still in history: `comm -13 <(git ls-files | sort) <(git log --all --name-only --format= | sort -u)` lists them; look at any that might hold data (`.env`, exports, dumps).

## 3. Scan issues, pull requests and comments

```sh
T=$(mktemp)
gh api --paginate "$R/issues?state=all&per_page=100" --jq '.[] | "#\(.number) \(.title)\n\(.body // "")"' > "$T"
gh api --paginate "$R/issues/comments?per_page=100" --jq '.[] | .html_url + "\n" + .body' >> "$T"
gh api --paginate "$R/pulls/comments?per_page=100" --jq '.[] | .html_url + "\n" + .body' >> "$T"
```

(The first covers pull requests too: GitHub lists them as issues.)

Search `"$T"` with the same patterns and terms as step 2.

**Edit history:** an edited issue, pull request or comment keeps every earlier version, visible to anyone once public. List what was edited and search the old versions:

```sh
gh api graphql --paginate -F owner=<owner> -F repo=<name> -f query='
query($owner: String!, $repo: String!, $endCursor: String) {
  repository(owner: $owner, name: $repo) {
    issues(first: 50, after: $endCursor) {
      pageInfo { hasNextPage endCursor }
      nodes { url lastEditedAt userContentEdits(first: 20) { nodes { diff } }
        comments(first: 50) { nodes { url lastEditedAt userContentEdits(first: 20) { nodes { diff } } } } }
    }
  }
}' --jq '.data.repository.issues.nodes[] | (select(.lastEditedAt) | .url + "\n" + ([.userContentEdits.nodes[].diff] | join("\n"))), (.comments.nodes[] | select(.lastEditedAt) | .url + "\n" + ([.userContentEdits.nodes[].diff] | join("\n")))'
```

Run it again with `pullRequests` in place of `issues`. A finding in an old version is removed through the "edited" menu on that item → the revision → **Delete revision from history**; the API can't do it, so the user does.

**Workflow logs** become public too. GitHub masks repository secrets in them, but anything printed by a script (an email, a config dump) isn't; if the scans turned something up, look at recent runs, and old runs can be deleted (`gh run delete`).

## 4. Report the findings

A short list: each finding, where it is (commit, file, issue, revision), and what to do. A real secret is **rotated first** (revoked where it was issued, replaced in repository secrets), then removed; removing it from history alone isn't enough, since it may be copied already. Rewriting history (to drop a file or an address) breaks every existing clone, tag and pull request link: suggest it only for something that truly can't be public, and as its own decision.

Nothing is public until the user has dealt with every finding or decided to leave it.

## 5. Fill what a public repository needs

Run the `onboard` skill's audit, public rows: `LICENSE` (ask which licence; anything kept out of it, such as the name and logo, goes in the README), `SECURITY.md`, `CODE_OF_CONDUCT.md`, `.github/CODEOWNERS`, the security-report link in the issue chooser, the README's licence section, and CONTRIBUTING's welcome paragraph. Each area is an issue and a pull request. Also look for anything that only worked while private: a badge URL with a token (Codecov and others serve public repositories without one).

## 6. Check deploy previews

If a service builds previews of pull requests (Cloudflare, Vercel, Netlify and others connected to the repository), a pull request from a fork would run its own code in that build, with the service's settings and secrets. Check in the service's dashboard that fork pull requests don't get preview builds, or need approval. This can't be checked from here; ask the user to confirm.

## 7. The switch is the owner's

Tell the user what's done and what isn't, then: **Settings → General → Danger Zone → Change visibility → Public**. Never run it yourself. On the free plan, this also lifts the Actions minutes limit for this repository, and allows a branch ruleset.

## 8. Straight after

Show, then run, the public part of the `onboard` skill's settings (step 4): private vulnerability reporting, fork pull request approval, secret scanning and push protection, and the `main` ruleset. Then check a pull request goes through: CI, the format check and the Claude review.

## Never

- Change the visibility, rewrite history or delete runs without the user's explicit yes for that specific action.
- Treat a secret as handled because it's no longer in the code: it's handled when it's rotated.
- Print a found secret in full in the conversation; name where it is and its kind.
