# Templates

The files the `onboard` skill writes into a repository. `base/` is for every repository; `public/` is added for a public one (or when a private one goes public), and its files replace `base/` ones at the same path (`.github/ISSUE_TEMPLATE/config.yml` gains the private security-report link). Paths inside mirror the repository: `base/.github/workflows/test.yml` becomes `.github/workflows/test.yml`.

For a public repository, the base docs also get, in their own words for the project:

- **README:** a "Licence" section (the licence; anything kept out of it, such as the name and logo) and a "Security problems: `SECURITY.md`" line under Docs.
- **CONTRIBUTING.md:** a first paragraph saying contributions are welcome under the licence, everyone follows the [Code of Conduct](CODE_OF_CONDUCT.md), and security problems go through `SECURITY.md`, not issues.

The licence template is MIT; another licence means its own text, chosen with the owner.

The skill fills the placeholders below (`{{UPPER_CASE}}`; GitHub's own `${{ … }}` expressions in the workflows stay as they are), then reads every file through once: the result must read as written for that project, with no placeholder or "TODO" left. Sections that don't apply (no releases yet, no scopes) are cut, not left empty.

## Placeholders

| Placeholder | What | Example |
| --- | --- | --- |
| `{{OWNER}}` | GitHub owner | `adamchrabaszcz` |
| `{{REPO}}` | Repository name | `abc` |
| `{{PROJECT}}` | Display name, as written in prose | `ABC` |
| `{{DESCRIPTION}}` | One sentence: what it is and for whom | `A tool that …` |
| `{{STACK}}` | Language and runtime, in words | `Node 24, no dependencies` |
| `{{SETUP_COMMAND}}` | Install or build once after cloning; cut the line if none | `npm ci` |
| `{{TEST_COMMAND}}` | The full check a contributor runs | `npm test` |
| `{{TEST_STEPS}}` | The `test` job's steps after checkout (YAML, indented 6 spaces); see below | |
| `{{ECOSYSTEM}}` | Dependabot's package ecosystem for the stack; cut that entry if none | `npm` |
| `{{SCOPES}}` | Commit scopes, comma-separated, or cut the line | `` `cli`, `core` `` |
| `{{CODE_RULES}}` | AGENTS.md bullets: the project's rules that bite (bullets) | |
| `{{REVIEW_POINTS}}` | AGENTS.md "Code review" bullets: what CI can't see here (bullets) | |
| `{{YEAR}}`, `{{COPYRIGHT_HOLDER}}` | For `LICENSE` | `2026`, `Ada Example` |
| `{{DATE}}` | Today, for ADRs 0001–0003 | `2026-11-24` |

`{{CODE_RULES}}` and `{{REVIEW_POINTS}}` come from reading the code (or, for a new repository, from what the owner says it will be). Start small: three to five bullets each, the things that would hurt if broken. They grow with the project.

## `{{TEST_STEPS}}` by stack

```yaml
# Node
      - uses: actions/setup-node@949feb2413d6458794dcd2491c4babbbce0c15c1 # v7.1.0
        with:
          node-version: 24
          cache: npm
      - run: npm ci
      - run: npm test
# Nothing to test yet (a new repository): keep CI green until there is.
      - run: echo "No tests yet; add them with the first code."
```

For other stacks, use the stack's official setup action, pinned to a commit SHA with the version in a comment, then the install and test commands.

## Pinned versions

Every `uses:` in these templates is pinned to a commit SHA with its version in a comment. In a repository, Dependabot keeps them current. Here, nothing does: when an action used in these templates releases, update the SHA and comment in every file, checking the SHA against the tag (`gh api repos/<owner>/<action>/commits/<tag> --jq .sha`).

| Action | Version |
| --- | --- |
| `actions/checkout` | v7.0.1 |
| `actions/setup-node` | v7.1.0 |
| `anthropics/claude-code-action` | v1.0.286 |
| `adamchrabaszcz/github-steward-actions` (`format`, `close-parent`, `release`) | v1.0.0 |
