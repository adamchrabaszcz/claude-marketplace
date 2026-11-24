# Claude Marketplace

A collection of Claude Code plugins for various frameworks and tools.

## Available Plugins

### symfony-developer

A comprehensive Claude Code plugin for Symfony 7.x development featuring:

- **symfony-7-developer agent**: An expert Symfony developer agent with deep knowledge of Symfony 7.x architecture, Doctrine ORM, testing, and best practices

**Installation:**
```
/plugin marketplace add adamchrabaszcz/claude-marketplace
/plugin install symfony-developer@adams-claude-marketplace
```

Renamed from `claude-symfony-plugin`, because names starting with `claude-` are reserved for Anthropic's own plugins. If you installed it under the old name, uninstall that (`/plugin uninstall claude-symfony-plugin@adams-claude-marketplace`) and install `symfony-developer`.

### github-steward

Sets GitHub repositories up and keeps work in them tidy. Language-agnostic; uses the `gh` CLI.

- **deliver** skill: the working loop for any change.
  - A plan first, approved by you.
  - An issue for every task, and one branch and one small PR per change, each from the latest `main` and mergeable on its own.
  - Checks and a self-review before committing; Conventional Commits; PRs that close their issue.
  - A test merge of all open PRs before reporting back.
- **onboard** skill: "let's start working on ABC" creates the repository, private or public, already set up: AGENTS.md, CONTRIBUTING, docs and decision records, issue and PR templates, CI, the format check, releases, the Claude review, Dependabot and the GitHub settings. It also audits an existing repository and fills what's missing.
- **go-public** skill: takes a private repository public safely. It scans the whole history, every issue, pull request and comment (edit history included) for secrets and personal data, fills what a public repository needs, and turns on the public settings once you've switched.

**Needs:**
- The [`gh`](https://cli.github.com) CLI, logged in with the `repo` and `workflow` scopes.
- For the Claude review in the repositories it sets up: the [Claude GitHub App](https://github.com/apps/claude) covering them, and a token from `claude setup-token` set as each repository's `CLAUDE_CODE_OAUTH_TOKEN` secret. The skill asks for both when it gets there; the token is typed at a prompt, never pasted into the chat.
- On GitHub's free plan, private repositories share 2,000 Actions minutes a month (CI and the review both count), and can't have a branch ruleset. Public repositories have neither limit.

**Installation:**
```
/plugin marketplace add adamchrabaszcz/claude-marketplace
/plugin install github-steward@adams-claude-marketplace
```

## Structure

```
claude-marketplace/
├── plugins/
│   ├── github-steward/           # GitHub repo setup and workflow (skills)
│   └── symfony-developer/        # Symfony 7.x development plugin
│       ├── .claude-plugin/
│       │   └── plugin.json
│       └── agents/
│           └── symfony-7-developer.md
└── README.md
```

## Contributing

Contributions are welcome; [`CONTRIBUTING.md`](CONTRIBUTING.md) has the workflow and how to add a plugin, and [`AGENTS.md`](AGENTS.md) the short version for coding agents. Everyone taking part follows the [Code of Conduct](CODE_OF_CONDUCT.md). Security problems: [`SECURITY.md`](SECURITY.md), reported privately, not in issues.

## License

[MIT](LICENSE): use, copy, change and share the plugins, including in your own work, as long as the licence and copyright notice stay with them.
