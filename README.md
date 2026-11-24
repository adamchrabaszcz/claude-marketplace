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
- **onboard** skill (coming): checks a repository and sets up what's missing:
  - AGENTS.md, docs, CONTRIBUTING.md and README;
  - labels, sidebar and branch protection;
  - CI with coverage, and releases from Conventional Commits.

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
